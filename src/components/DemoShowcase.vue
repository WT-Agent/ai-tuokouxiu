<template>
  <NomadsShowcase @apply-template="$emit('use-sample', $event.prompt, $event.prompt)" />
</template>

<script setup lang="ts">
import NomadsShowcase from './NomadsShowcase.vue';

defineEmits<{
  (e: 'use-sample', sampleTopic: string, sampleDestination: string): void;
}>();
</script>
