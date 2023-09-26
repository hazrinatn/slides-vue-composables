<script setup>
import { useMouse } from './mouse.js'

const { x, y } = useMouse()
</script>

<template>
    <div
    border="l main"
    hover:bg="gray-400 opacity-20">
    Mouse position is at: {{ x }}, {{ y }}</div>

</template>

<style>

</style>