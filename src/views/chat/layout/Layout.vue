<!--
 * @Author: mjjh
 * @LastEditTime: 2023-04-15 22:10:36
 * @FilePath: \chatgpt-shuowen\src\views\chat\layout\Layout.vue
 * @Description: 页面布局文件
-->
<script setup lang='ts'>
import { computed, h, onMounted } from 'vue'
import { NButton, NLayout, NLayoutContent, NModal, NTabPane, NTabs, useMessage, useNotification } from 'naive-ui'
import { useRouter } from 'vue-router'
import Sider from './sider/index.vue'
import login from './login.vue'
import register from './register.vue'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { useAppStore, useAuthStore, useChatStore, useUserStore } from '@/store'

const router = useRouter()
const appStore = useAppStore()
const chatStore = useChatStore()
const authStore = useAuthStore()
const userStore = useUserStore()
const message = useMessage()
const notification = useNotification()

router.replace({ name: 'Chat', params: { uuid: chatStore.active } })

const { isMobile } = useBasicLayout()

const collapsed = computed(() => appStore.siderCollapsed)

const needPermission = computed(() => !authStore.token)

onMounted(() => {
  handleClick2()
})

// 获取屏幕尺寸适配移动端
const getMobileClass = computed(() => {
  if (isMobile.value)
    return ['rounded-none', 'shadow-none']
  return ['border', 'rounded-md', 'shadow-md', 'dark:border-neutral-800']
})

const getContainerClass = computed(() => {
  return [
    'h-full',
    { 'pl-[260px]': !isMobile.value && !collapsed.value },
  ]
})

function handleClick2() {
  let markAsRead = false
  const n = notification.create({
    title: 'AIGC V1.0使用说明~',
    content: '本网站基于GPT-3.5T，只可用于学习或解答问题\n请遵守法律法规,严禁搜索和传播敏感违法内容\n本网站还在持续开发中，大佬若有兴趣可一起探究\nwechat：1939430189',
    meta: '2023-7-10 星期一',
    action: () =>
      h(
        NButton,
        {
          text: true,
          type: 'primary',
          onClick: () => {
            markAsRead = true
            n.destroy()
          },
        },
        {
          default: () => '已读',
        },
      ),
    onClose: () => {
      if (!markAsRead) {
        message.warning('请设为已读')
        return false
      }
    },
  })
}

// 页面加载请求用户信息
if (!needPermission.value)
  userStore.getUserData()
</script>

<template>
  <div class="h-full dark:bg-[#24272e] transition-all" :class="[isMobile ? 'p-0' : 'p-4']">
    <div class="h-full overflow-hidden" :class="getMobileClass">
      <NLayout class="z-40 transition" :class="getContainerClass" has-sider>
        <Sider />
        <NLayoutContent class="h-full">
          <RouterView v-slot="{ Component, route }">
            <component :is="Component" :key="route.fullPath" />
          </RouterView>
        </NLayoutContent>
      </NLayout>
    </div>
    <NModal :show="needPermission" style="width: 90%; max-width: 640px">
      <div class="p-10 bg-white rounded dark:bg-slate-800">
        <div class="space-y-4">
          <NTabs default-value="login" size="large" animated>
            <NTabPane name="login" tab="登录">
              <login />
            </NTabPane>
            <NTabPane name="register" tab="注册">
              <register />
            </NTabPane>
          </NTabs>
        </div>
      </div>
    </NModal>
  </div>
</template>
