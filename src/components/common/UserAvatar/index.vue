<script setup lang='ts'>
import { computed, onMounted, ref } from 'vue'
import { NAvatar } from 'naive-ui'
import { useAuthStore, useUserStore } from '@/store'
import defaultAvatar from '@/assets/avatar.jpg'
import { isString } from '@/utils/is'
import { fetchChatConfig } from '@/api'

interface ConfigState {
  timeoutMs?: number
  reverseProxy?: string
  apiModel?: string
  socksProxy?: string
  httpsProxy?: string
  balance?: string
}

const userStore = useUserStore()

const authStore = useAuthStore()

const userInfo = computed(() => userStore.userInfo)

const needPermission = computed(() => !!authStore.session?.auth && !authStore.token)

const config = ref<ConfigState>()

async function fetchConfig() {
  const { data } = await fetchChatConfig<ConfigState>()
  if (data.balance === '-')
    data.balance = '无效'
  config.value = data
}

onMounted(() => {
  if (!needPermission.value)
    fetchConfig()
})
</script>

<template>
  <div class="flex items-center overflow-hidden">
    <div class="w-10 h-10 overflow-hidden rounded-full shrink-0">
      <template v-if="isString(userInfo.avatar) && userInfo.avatar.length > 0">
        <NAvatar
          size="large"
          round
          :src="userInfo.avatar"
          :fallback-src="defaultAvatar"
        />
      </template>
      <template v-else>
        <NAvatar size="large" round :src="defaultAvatar" />
      </template>
    </div>
    <div class="flex-1 min-w-0 ml-2">
      <h2 class="overflow-hidden font-bold text-md text-ellipsis whitespace-nowrap">
        {{ userInfo.name ?? 'YYのChatGPT公益站' }}
      </h2>
      <!-- <p class="overflow-hidden text-xs text-gray-500 text-ellipsis whitespace-nowrap">
        公益{{ $t("setting.balance") }}：${{ config?.balance ?? '-' }}
      </p> -->
      <p class="overflow-hidden text-xs text-gray-500 text-ellipsis whitespace-nowrap">
        <span
          v-if="isString(userInfo.description) && userInfo.description !== ''"
          v-html="userInfo.description"
        />
      </p>
    </div>
  </div>
</template>
