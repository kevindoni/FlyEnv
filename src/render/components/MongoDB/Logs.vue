<template>
  <div class="module-config">
    <el-card>
      <LogVM ref="log" :log-file="filepath" />
      <template #footer>
        <ToolVM :log="log" />
      </template>
    </el-card>
  </div>
</template>

<script lang="ts" setup>
  import { computed, ref } from 'vue'
  import LogVM from '@/components/Log/index.vue'
  import ToolVM from '@/components/Log/tool.vue'
  import { AppStore } from '@/store/app'
  import { join } from '@/util/path-browserify'

  const appStore = AppStore()
  const currentVersion = computed(() => {
    return appStore.config?.server?.mongodb?.current?.version
  })

  const log = ref()
  const filepath = computed(() => {
    if (!currentVersion?.value) {
      return ''
    }
    const v = currentVersion?.value?.split('.')?.slice(0, 2)?.join('.')
    return join(window.Server.MongoDBDir, `mongodb-${v}.log`)
  })
</script>
