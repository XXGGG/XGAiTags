<template>
</template>

<script lang="ts" setup>
import { useMessage } from 'naive-ui'

// content
const message = useMessage()
function show () {
    message.success('复制成功！')
}

defineExpose({show})
</script>

<style lang="scss" scoped></style>
