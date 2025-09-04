<template>
    <StringBuilder :key="updateComponent" v-bind="content" />
    <IncDecDemo />
</template>

<script setup>
import StringBuilder from './string-builder/StringBuilder.vue';
import IncDecDemo from './string-builder/IncDecDemo.vue';
import { watch, ref } from 'vue';
const props = defineProps({
    content: { type: Object, required: true },
});

const updateComponent = ref(0);
watch(
    () => props.content,
    () => (updateComponent.value += 1),
    { deep: true }
);
</script>
