<template>
  <div>
    <modal-factory />
    <router-view />
  </div>
</template>

<script>
import { defineComponent, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { setCurrentUser } from './store/user'
import ModalFactory from './components/ModalFactory'
import services from './services'

export default defineComponent({
  components: {
    ModalFactory
  },
  setup () {
    const route = useRoute()
    const router = useRouter()

    watch(
      () => route.path,
      async () => {
        if (route.meta.hasAuth) {
          const token = window.localStorage.getItem('token')

          if (!token) {
            router.push({ name: 'Home' })
            return
          }

          const { data } = await services.users.getMe()
          setCurrentUser(data)
        }
      }
    )
  }
})
</script>
