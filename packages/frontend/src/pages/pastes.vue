<template>
  <div>
    <Header />
    <div class="px-5 py-6">
      <h1 class="font-mono text-white text-2xl">Your Pastes</h1>

      <div
        class="mt-6 flex flex-col gap-y-4 pr-5 overflow-y-scroll"
        style="height: 74vh"
      >
        <nuxt-link
          v-for="(paste, i) in state.pastes"
          :key="i"
          :to="`/${paste.id}`"
          class="rounded-2xl flex flex-col border px-4 py-2 border-gray-700 text-white font-mono"
        >
          <span
            class="truncate break-all overflow-ellipsis overflow-hidden tracking-tighter max-w-4xl"
          >
            {{ paste.content }}
          </span>
          <span class="text-xs text-gray-400">
            {{ new Date(paste.createdAt).toLocaleString() }} &ndash;
            <span
              class="text-gray-200 border-b border-gray-200 cursor-pointer"
              @click.prevent="() => removePaste({ id: paste.id })"
            >
              Delete
            </span>
          </span>
        </nuxt-link>
        <div v-if="state.pastes.length < 1" class="text-white font-mono">
          You have no pastes yet.
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {
  computed,
  defineComponent,
  reactive,
  useContext,
} from '@nuxtjs/composition-api'
import { PasteDto } from '@snipcode/backend/src/http/dto/db/pasteDto'
import { Paste } from '@snipcode/backend/src/http/schemas'
import socketSend from '@snipcode/backend/src/ws/helpers/socketSend'
import { CreateWebSocket } from '../api/ws/createWebSocket'
import Header from '../components/layout/Header.vue'

export default defineComponent({
  components: { Header },
  middleware: 'requiredAuth',
  setup() {
    const state = reactive({
      pastes: [] as PasteDto[],
    })

    const { $accessor } = useContext()

    const user = computed(() => $accessor.user.user)

    // Loads pastes from User Store
    state.pastes = user.value ? user.value.pastes ?? [] : []

    if (!$accessor.socket.socket)
      throw new Error('WebSockets are not supported.')

    // WebSocket connection in store
    const [socket, emitter]: CreateWebSocket = $accessor.socket.socket

    const removePaste = async (data: Paste.ById['Params']) => {
      try {
        socketSend(socket, {
          action: 'paste_delete',
          data,
        })
        $accessor.setTimedAlert({ value: 'Paste deleted.', time: 1000 })
      } catch (_) {}
    }

    socket.addEventListener('open', () => {
      socketSend(socket!, { action: 'paste_fetch' })
    })

    emitter.on('fetch_pastes', (msg) => {
      state.pastes = msg.data.pastes
    })

    emitter.on('paste_create', (msg) => {
      state.pastes = [msg.data.paste, ...state.pastes]
    })

    emitter.on('paste_delete', (msg) => {
      state.pastes = state.pastes.filter((el) => el.id !== msg.data.paste.id)
    })

    return {
      state,
      removePaste,
    }
  },
})
</script>
