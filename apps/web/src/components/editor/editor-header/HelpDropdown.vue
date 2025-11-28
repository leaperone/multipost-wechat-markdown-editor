<script setup lang="ts">
const props = withDefaults(defineProps<{
  asSub?: boolean
}>(), {
  asSub: false,
})

const emit = defineEmits([`openAbout`])

const { asSub } = toRefs(props)

function openAboutDialog() {
  emit(`openAbout`)
}
</script>

<template>
  <!-- 作为 MenubarSub 使用 -->
  <MenubarSub v-if="asSub">
    <MenubarSubTrigger>
      帮助
    </MenubarSubTrigger>
    <MenubarSubContent align="start">
      <MenubarItem @click="openAboutDialog()">
        <span>关于</span>
      </MenubarItem>
    </MenubarSubContent>
  </MenubarSub>

  <!-- 作为 MenubarMenu 使用（默认） -->
  <MenubarMenu v-else>
    <MenubarTrigger>帮助</MenubarTrigger>
    <MenubarContent align="start">
      <MenubarItem @click="openAboutDialog()">
        <span>关于</span>
      </MenubarItem>
    </MenubarContent>
  </MenubarMenu>
</template>
