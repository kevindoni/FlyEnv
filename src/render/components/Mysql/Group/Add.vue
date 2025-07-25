<template>
  <el-dialog
    v-model="show"
    :title="item?.dataDir ? I18nT('base.edit') : I18nT('base.add')"
    width="600px"
    :destroy-on-close="true"
    class="host-edit new-project"
    @closed="closedFn"
  >
    <template #default>
      <div class="main-wapper">
        <div class="main p-5">
          <div class="path-choose mt-5 mb-10">
            <el-select
              v-model="form.path"
              class="w-full"
              :class="{ error: errs?.path }"
              placeholder="MySQL Version"
            >
              <template v-for="(item, _index) in mysqlVersion" :key="_index">
                <el-option :label="`${item.version}-${item.bin}`" :value="item.path"></el-option>
              </template>
            </el-select>
          </div>
          <div class="path-choose my-5">
            <input
              v-model.trim="form.port"
              type="text"
              class="input"
              :class="{ error: errs?.port }"
              placeholder="TCP Port"
            />
          </div>
          <div class="path-choose my-5">
            <input
              type="text"
              class="input"
              :class="{ error: errs?.dataDir }"
              placeholder="MySQL Data Directory"
              readonly="true"
              :value="form.dataDir"
            />
            <div class="icon-block" @click="chooseRoot()">
              <yb-icon
                :svg="import('@/svg/folder.svg?raw')"
                class="choose"
                width="18"
                height="18"
              />
            </div>
          </div>
        </div>
      </div>
    </template>
    <template #footer>
      <div class="dialog-footer">
        <el-button :disabled="running" @click="show = false">{{ I18nT('base.cancel') }}</el-button>
        <el-button :loading="running" :disabled="running" type="primary" @click="doSave">{{
          I18nT('base.confirm')
        }}</el-button>
      </div>
    </template>
  </el-dialog>
</template>
<script lang="ts" setup>
  import { computed, ref, watch } from 'vue'
  import { AsyncComponentSetup } from '@/util/AsyncComponent'
  import { I18nT } from '@lang/index'
  import { uuid } from '@/util/Index'
  import { BrewStore } from '@/store/brew'
  import { MessageSuccess } from '@/util/Element'
  import type { MysqlGroupItem } from '@shared/app'
  import { MysqlStore } from '../mysql'
  import { join } from '@/util/path-browserify'
  import { dialog } from '@/util/NodeFn'

  const { show, onClosed, onSubmit, closedFn } = AsyncComponentSetup()

  const props = defineProps<{
    item: MysqlGroupItem
  }>()

  const mysqlStore = MysqlStore()
  const brewStore = BrewStore()
  const running = ref(false)
  const form = ref({
    id: uuid(8).toUpperCase(),
    path: '',
    port: undefined,
    dataDir: ''
  })

  Object.assign(form.value, props.item)
  form.value.path = props?.item?.version?.path ?? ''
  if (!form.value.dataDir) {
    form.value.dataDir = join(window.Server.MysqlDir!, `group/mysql-group-${form.value.id}`)
  }

  const errs = ref({
    path: false,
    port: false,
    dataDir: false
  })

  const mysqlVersion = computed(() => {
    return brewStore.module('mysql').installed
  })

  watch(
    form,
    () => {
      let k: keyof typeof errs.value
      for (k in errs.value) {
        errs.value[k] = false
      }
    },
    {
      immediate: true,
      deep: true
    }
  )

  const checkItem = () => {
    errs.value.path = !form.value.path
    errs.value.port = !form.value.port
    errs.value.dataDir = !form.value.dataDir

    let k: keyof typeof errs.value
    for (k in errs.value) {
      if (errs.value[k]) {
        return false
      }
    }
    return true
  }

  const chooseRoot = () => {
    if (running?.value) {
      return
    }
    dialog
      .showOpenDialog({
        properties: ['openDirectory', 'createDirectory', 'showHiddenFiles']
      })
      .then(({ canceled, filePaths }: any) => {
        if (canceled || filePaths.length === 0) {
          return
        }
        const [path] = filePaths
        form.value.dataDir = path
      })
  }

  const doSave = () => {
    if (!checkItem() || running?.value) {
      return
    }
    running.value = true
    const version: any = mysqlVersion.value.find((m) => m.path === form.value.path)!
    if (!props?.item?.id) {
      mysqlStore.all.push({
        id: form.value.id,
        version: JSON.parse(JSON.stringify(version)),
        port: form.value.port!,
        dataDir: form.value.dataDir
      })
    } else {
      const item = mysqlStore.all.find((f) => f.id === props.item.id)
      if (item) {
        const running = item.version.running
        mysqlStore.stop(item)
        item.version = JSON.parse(JSON.stringify(version))
        item.port = form.value.port!
        item.dataDir = form.value.dataDir
        mysqlStore.save().then()
        if (running) {
          mysqlStore.start(item).then()
        }
      }
    }

    mysqlStore.save()
    MessageSuccess(I18nT('base.success'))
    running.value = false
    show.value = false
  }

  defineExpose({
    show,
    onSubmit,
    onClosed
  })
</script>
