<template>
  <div class="soft-index-panel main-right-panel">
    <el-radio-group v-model="tab" class="mt-3">
      <template v-for="(item, _index) in tabs" :key="_index">
        <el-radio-button :label="item" :value="_index"></el-radio-button>
      </template>
    </el-radio-group>
    <div class="main-block">
      <Service
        v-if="tab === 0"
        title="Ruby"
        type-flag="ruby"
        :fetch-data-when-create="true"
      ></Service>
      <Manager
        v-else-if="tab === 1"
        type-flag="ruby"
        :has-static="false"
        title="Ruby"
        url="https://github.com/oneclick/rubyinstaller2/releases"
      ></Manager>
      <ProjectIndex
        v-else-if="tab === 2"
        :title="`Ruby ${I18nT('base.projects')}`"
        :type-flag="'ruby'"
      >
        <template #openin="{ row }">
          <li @click.stop="Project.openPath(row.path, 'RubyMine')">
            <yb-icon :svg="import('@/svg/rubymine.svg?raw')" width="13" height="13" />
            <span class="ml-3">{{ I18nT('nodejs.openIN') }} RubyMine</span>
          </li>
        </template>
      </ProjectIndex>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import Service from '@/components/ServiceManager/base.vue'
  import Manager from '../VersionManager/index.vue'
  import { AppModuleSetup } from '@/core/Module'
  import { I18nT } from '@lang/index'
  import ProjectIndex from '@/components/LanguageProjects/index.vue'
  import { Project } from '@/util/Project'

  const { tab } = AppModuleSetup('ruby')
  const tabs = [
    I18nT('base.service'),
    I18nT('base.versionManager'),
    `Ruby ${I18nT('base.projects')}`
  ]
</script>
