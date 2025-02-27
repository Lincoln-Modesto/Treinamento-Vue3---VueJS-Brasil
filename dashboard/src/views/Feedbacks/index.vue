<template>
 <div>
   <div class="flex justify-center w-full h-28 bg-brand-main">
    <header-logged />
  </div>

  <div class="flex flex-col items-center justify-center h-64 bg-brand-gray">
    <h1 class="text-4xl font-black text-center text-gray-800">Feedbacks</h1>
    <p class="text-lg text-center text-gray-800 font-regular">
      Detalhes de todos os feedbacks recebidos
    </p>
  </div>

  <div class="flex justify-center w-full pb-20">
    <div class="w-4/5 max-w-6xl py-10 grid grid-cols-4 gap-2">
      <div>
        <h1 class="text-3xl font-black text-brand-darkgray">Listagem</h1>

        <suspense>
          <template #default>
            <filters
            @select="changeFeedbacksType"
              class="mt-8 animate__animated animate__fadeIn animated__faster"
            />
          </template>
          <template #fallback>
            <filters-loading class="mt-8" />
          </template>
        </suspense>
      </div>
      <div class="px-10 py-20 col-span-3">
        <p
          class="text-lg text-center text-gray-800 font-regular"
          v-if="state.hasErro"
        >
          Aconteceu um erro ao carregar os feedbacks
        </p>
        <p
          class="text-lg text-center text-gray-800 font-regular"
          v-if="!state.feedbacks.length && !state.isLoading && !state.isLoadingFeedbacks && !state.hasError"
        >
          Ainda não há nenhum feedback recebido
        </p>

        <feedback-card-loading v-if="state.isLoading || state.isLoadingFeedbacks" />
        <feedback-card
          v-for="(feedback, index) in state.feedbacks"
          :key="index"
          v-else
          :is-opened="index === 0"
          :feedback="feedback"
          class="mb-8"
        />
        <feedback-card-loading v-if="state.isLoadingMoreFeedbacks" />
      </div>
    </div>
  </div>
 </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, onUnmounted, reactive, onErrorCaptured } from 'vue'
import HeaderLogged from '../../components/HeaderLogged/index.vue'
import Filters from './Filters.vue'
import FiltersLoading from './FiltersLoading.vue'
import FeedbackCard from '../../components/FeedbackCard/index.vue'
import FeedbackCardLoading from '../../components/FeedbackCard/loading.vue'
import services from '../../services'

export default defineComponent({
  components: {
    HeaderLogged,
    Filters,
    FiltersLoading,
    FeedbackCardLoading,
    FeedbackCard
  },
  setup () {
    const state = reactive({
      isLoading: false,
      isLoadingFeedbacks: false,
      isLoadingMoreFeedbacks: false,
      hasErro: false,
      feedbacks: [],
      currentFeedbackType: '',
      pagination: {
        limit: 5,
        offset: 0,
        total: 0
      }
    })

    onErrorCaptured(handleErrors)

    onMounted(() => {
      fetchFeedbacks()
      window.addEventListener('scroll', handleScroll, false)
    })

    onUnmounted(() => {
      window.removeEventListener('scroll', handleScroll, false)
    })

    async function handleScroll () {
      const isBottomOfWindow = Math.ceil(
        document.documentElement.scrollTop + window.innerHeight
      ) >= document.documentElement.scrollHeight

      if (state.isLoading || state.isLoadingMoreFeedbacks) return
      if (!isBottomOfWindow) return
      if (state.feedbacks.length >= state.pagination.total) return

      try {
        state.isLoadingMoreFeedbacks = true
        const { data } = await services.feedbacks.getAll({
          ...state.pagination,
          type: state.currentFeedbackType,
          offset: (state.pagination.offset + 5)
        })

        if (data.results.length) {
          // @ts-ignore
          state.feedbacks.push(...data.results)
        }

        state.isLoadingMoreFeedbacks = false
        state.pagination = data.pagination
      } catch (error) {
        state.isLoadingMoreFeedbacks = false
        handleErrors(error)
      }
    }

    function handleErrors (error: boolean): void {
      state.isLoading = false
      state.isLoadingFeedbacks = false
      state.isLoadingMoreFeedbacks = false
      state.hasErro = !!error
    }

    async function changeFeedbacksType (type: string): Promise<void> {
      try {
        state.isLoadingFeedbacks = true
        state.pagination.limit = 5
        state.pagination.offset = 0
        state.currentFeedbackType = type

        const { data } = await services.feedbacks.getAll({
          type,
          ...state.pagination
        })

        state.feedbacks = data.results
        state.pagination = data.pagination
        state.isLoadingFeedbacks = false
      } catch (error) {
        handleErrors(error)
      }
    }

    async function fetchFeedbacks (): Promise<void> {
      try {
        state.isLoading = true
        const { data } = await services.feedbacks.getAll({
          ...state.pagination,
          type: state.currentFeedbackType
        })

        state.feedbacks = data.results
        state.pagination = data.pagination
        state.isLoading = false
      } catch (error) {
        handleErrors(error)
      }
    }

    return {
      state,
      changeFeedbacksType
    }
  }
})
</script>

<style></style>
