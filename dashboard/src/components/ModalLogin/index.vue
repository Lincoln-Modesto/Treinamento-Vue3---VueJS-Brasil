<template>
  <div class="flex justify-between">
    <h1 class="text-4xl font-black text-gray-800">Entre na sua conta</h1>
    <button class="text-4xl text-gray-600 focus:outline-none" @click="close">
      &times;
    </button>
  </div>

  <div class="mt-16">
    <form @submit.prevent="handleSubmit">
      <label class="block">
        <span class="text-lg font-medium text-gray-800">E-mail</span>
        <input
          type="email"
          class="
            block
            w-full
            px-4
            py-3
            mt-1
            text-large
            bg-gray-100
            border-2 border-transparent
            rounded
          "
          :class="{ 'border-brand-danger': !!state.email.errorMessage }"
          placeholder="lincoln@gmail.com"
          v-model="state.email.value"
        />
        <span
          v-if="!!state.email.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.email.errorMessage }}
        </span>
      </label>

      <label class="block mt-9">
        <span class="text-lg font-medium text-gray-800">Senha</span>
        <input
          type="password"
          class="
            block
            w-full
            px-4
            py-3
            mt-1
            text-large
            bg-gray-100
            border-2 border-transparent
            rounded
          "
          :class="{ 'border-brand-danger': !!state.password.errorMessage }"
          placeholder="********"
          v-model="state.password.value"
        />
        <span
          v-if="!!state.password.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.password.errorMessage }}
        </span>
      </label>

      <button
        class="
          px-8
          py-3
          mt-10
          text-2xl
          font-bold
          text-white
          rounded-full
          bg-brand-main
          focus:outline-none
          transition-all
          duration-150
        "
        :disabled="state.isLoading"
        type="submit"
        :class="{ 'opacity-50': state.isLoading }"
      >
        Entrar
      </button>
    </form>
  </div>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue'
import { validateEmptyAndLength3, validateEmptyAndEmail } from '../../utils/validators'
import { useField } from 'vee-validate'
import { useRouter } from 'vue-router'
import { useToast } from 'vue-toastification'
import useModal from '../../hooks/useModal'
import services from '../../services'

export default defineComponent({
  setup () {
    const modal = useModal()
    const router = useRouter()
    const toast = useToast()

    // eslint-disable-next-line @typescript-eslint/ban-ts-comment
    // @ts-ignore
    const { value: emailValue, errorMessage: emailErrorMessage } = useField('email', validateEmptyAndEmail)

    // eslint-disable-next-line @typescript-eslint/ban-ts-comment
    // @ts-ignore
    const { value: passwordValue, errorMessage: passwordErrorMessage } = useField('password', validateEmptyAndLength3)

    const state = reactive({
      hasError: false,
      isLoading: false,
      email: {
        value: emailValue,
        errorMessage: emailErrorMessage
      },
      password: {
        value: passwordValue,
        errorMessage: passwordErrorMessage
      }
    })

    async function handleSubmit (): Promise<void> {
      try {
        toast.clear()
        state.isLoading = true

        const { data, errors } = await services.auth.login({
          email: state.email.value as string,
          password: state.password.value as string
        })

        if (!errors) {
          window.localStorage.setItem('token', data.token)
          router.push({ name: 'Feedbacks' })
          state.isLoading = false
          modal.close()
          return
        }

        if (errors.status === 404) {
          toast.error('E-mail não encontrado')
        }
        if (errors.status === 401) {
          toast.error('E-mail/Senha inválidos')
        }
        if (errors.status === 400) {
          toast.error('Ocorreu um erro ao fazer o login')
        }

        state.isLoading = false
      } catch (error) {
        state.isLoading = false
        state.hasError = !!error
        toast.error('Ocorreu um erro ao fazer o login')
      }
    }

    return {
      state,
      close: modal.close,
      handleSubmit
    }
  }
})
</script>
