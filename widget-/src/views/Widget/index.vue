<template>
  <teleport to="body">
    <component
      @open-box="handleOpenBox"
      @close-box="handleCloseBox"
      :is="state.component"/>
  </teleport>
</template>

<script lang="ts">
import { defineComponent, reactive, onMounted } from 'vue'
import Standby from './Standby.vue'
import Box from './Box.vue'
import { useIframeControl } from '../../hooks/useIframe'

type State = {
  component: string
}

interface SetupReturn {
  state: State,
  handleOpenBox(): void
  handleCloseBox(): void
}

export default defineComponent({
  components: {
    Standby,
    Box
  },
  setup (): SetupReturn {
    const iframe = useIframeControl()
    const state = reactive<State>({
      component: 'Standby'
    })

    onMounted(() => {
      iframe.updateCoreValueOnStore()
    })

    function handleOpenBox (): void {
      iframe.notifyOpen()
      state.component = 'Box'
    }

    function handleCloseBox (): void {
      iframe.notifyClose()
      state.component = 'Standby'
    }

    return {
      state,
      handleCloseBox,
      handleOpenBox
    }
  }
})
</script>

<style>

</style>
