<template>
  <form class="w-full" @submit.prevent="login">
    <div class="pb-4">
      <div class="py-2">
        <text-input placeholder="Username" />
      </div>
      <div class="py-2">
        <text-input placeholder="Password" type="password" />
      </div>
    </div>
    <div class="flex items-center">
      <Button type="submit">Login</Button>
      <span
        class="text-purple-400 ml-4"
        v-if="store.options.auth.allowRegister"
      >
        or
        <inline-link is="router-link" to="/auth/register">
          register
        </inline-link>
      </span>
    </div>
  </form>
</template>

<script lang="ts">
import { defineComponent, reactive } from 'vue'
import Button from '../../components/elements/Button.vue'
import TextInput from '../../components/elements/form/TextInput.vue'
import InlineLink from '../../components/elements/InlineLink.vue'
import store, { setTitle } from '../../store'

export default defineComponent({
  components: { TextInput, Button, InlineLink },
  setup() {
    const state = reactive({
      loading: false,
    })

    setTitle('Login')

    const login = async () => {
      console.log('login')
    }

    return {
      state,
      store,
      login,
    }
  },
})
</script>
