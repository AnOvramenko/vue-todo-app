<script setup>
  const { todo } = defineProps(['todo']);
  const emit = defineEmits(['delete', 'update']);
</script>

<template>
  <div
    class="todo"
    :key="todo.id"
    :class="{ completed: todo.completed }"
  >
    <label class="todo__status-label">
      <input 
        type="checkbox" 
        class="todo__status" 
        :checked="todo.completed" 
        @change="emit('update', { ...todo, completed: !todo.completed })"
      />
    </label>

    <form v-if="false">
      <input
        type="text"
        class="todo__title-field"
        placeholder="Empty todo will be deleted"
      />
    </form>

    <template v-else>
      <span class="todo__title">{{ todo.title }}</span>
      <!-- Remove button appears only on hover -->
      <!-- <button type="button" class="todo__remove" @click="todos.splice(i, 1)"> -->
      <button type="button" class="todo__remove" @click="emit('delete')">
        ×
      </button>
    </template>

    <!-- overlay will cover the todo while it is being deleted or updated
          -->
    <div class="modal overlay" :class="{ 'is-active': false }">
      <div class="modal-background has-background-white-ter"></div>
      <div class="loader"></div>
    </div>
  </div>
</template>
