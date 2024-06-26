<script lang="ts" setup>
import { useConfigStore } from '@/store/config';
import { storeToRefs } from 'pinia';
import { Keymap } from "@/types/config";

const { enabledKeymaps, customParentKeymaps, options } = storeToRefs(useConfigStore())
const { saveConfig, translate } = useConfigStore()


// 初始颜色列表
const colors = ["pink", "blue", "purple", "purple", "deep-orange", "purple", "blue"];

function getHotkey(keymap: Keymap) {
  return keymap.parentID ? customParentKeymaps.value.find(k => k.id == keymap.parentID)!.hotkey : keymap.hotkey;
}

const getColor = (keymap: Keymap) => {
  // 获取首热键
  let hotkey = getHotkey(keymap)

  let hash = 0;
  for (let i = 0; i < hotkey.length; i++) {
    hash = hotkey.charCodeAt(i) + ((hash << 5) - hash);
  }

  return colors[Math.abs(hash) % colors.length];
}

const getIcon = (keymap: Keymap) => {
  let icon = "mdi-"
  let hotkey = getHotkey((keymap))

  // 判断是设置还是自定义热键
  if (hotkey == "settings") {
    return icon + "cog-outline"
  } else if (hotkey == "customHotkeys") {
    return icon + "keyboard-outline"
  } else if (hotkey == "capslockAbbr") {
    return icon + 'rocket-launch-outline'
  } else if (hotkey == "semicolonAbbr") {
    return icon + 'format-text-variant-outline'
  } else if (hotkey.toLowerCase().includes('button')) {
    return icon + 'cursor-default-outline'
  }

  // 去除热键中的符号
  hotkey = hotkey.replace(/^[^!#^+\w]/, '')
  // 获取首字母
  let key = hotkey.substring(0, 1)
  if (/[LlRr]/.test(key)) {
    key = hotkey.substring(1, 2)
  }
  key = key.toLowerCase()

  // 首字母不为热键修复符或字母
  if (/[^#!^+a-z0-9]/.test(key)) {
    return icon + "rhombus"
  }

  if (/\d/.test(key)) {
    return icon + "numeric-" + key + "-box-outline"
  }

  if (key == "!") {
    key = "a"
  } else if (key == "#") {
    key = "w"
  } else if (key == "^") {
    key = "c"
  } else if (key == "+") {
    key = "s"
  }

  if (/[a-zA-Z]/.test(key)) {
    return icon + "alpha-" + key + "-box"
  } else {
    return icon + "rhombus"
  }
}

</script>

<template>
  <v-navigation-drawer :permanent="true">
    <v-list-item to="/" :active="false">
      <template #prepend>
        <v-avatar size="70">
          <v-img src="@/assets/logo.png"></v-img>
        </v-avatar>
      </template>

      <v-list-item-title class="text-h5 font-500 h-1.2em">MyKeymap</v-list-item-title>
      <v-list-item-subtitle>{{ options.mykeymapVersion }}</v-list-item-subtitle>
    </v-list-item>

    <v-divider class="border-opacity-25"></v-divider>

    <v-list>
      <v-virtual-scroll :items="enabledKeymaps" height="calc(100vh - 158px)">
        <template #default="{ item: keymap, index }">
          <v-list-item :key="index" :value="keymap"
                       :to="keymap.id != 4 ? '/keymap/' + keymap.id : '/' + keymap.hotkey">
            <template #prepend>
              <v-icon :icon="getIcon(keymap)" :color="getColor(keymap)" size=28></v-icon>
            </template>
            <v-list-item-title>{{ keymap.name }}</v-list-item-title>
          </v-list-item>
        </template>
      </v-virtual-scroll>
    </v-list>

    <v-divider class="border-opacity-25"></v-divider>
    <v-btn class="ma-3" width="90%" color="blue" prepend-icon="mdi-content-save-outline" variant="outlined" @click="saveConfig">
      {{ translate('label:507') }}
    </v-btn>
  </v-navigation-drawer>
</template>

<style scoped>
.v-navigation-drawer :deep(i) {
  opacity: 0.94;
}
</style>
