<template>
  <div>
    <div class="w-3/4 h-3/4 ml-auto mr-auto mb-4">
      <select class="select">
        <option>Board 1</option>
        <option>Board 2</option>
        <option>Board 3</option>
      </select>
    </div>
    <div class="border-2 rounded-lg w-3/4 h-3/4 ml-auto mr-auto">
      <KanbanBoard />
    </div>
  </div>
</template>

<script setup lang="ts">
definePageMeta({
  layout: 'kanban'
})
</script>
