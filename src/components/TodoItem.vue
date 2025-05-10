<script setup>
import { nextTick, ref } from 'vue';

  //maybe need props instead of destruction todo
  const { todo } = defineProps(['todo']);
  const emit = defineEmits(['delete', 'update']);

  const editing = ref(false);
  const titleField = ref(null);
  const newTitle = ref(todo.title);

  const startEditing = async () => {
    newTitle.value = todo.title;
    editing.value = true;

    await nextTick();

    if (titleField.value) {
      titleField.value.focus();
    }
  };

  const rename = () => {
    if (!editing.value) return;

    editing.value = false;

    if (newTitle.value === todo.title) {
      return;
    }

    if (!newTitle.value) {
      emit('delete');
      return;
    }

    emit('update', {...todo, title: newTitle.value})
  }

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

    <form v-if="editing" @submit.prevent="rename">
      <input
        type="text"
        class="todo__title-field"
        placeholder="Empty todo will be deleted"
        ref="titleField"
        v-model.trim="newTitle"
        @keyup.escape="editing = false"
        @blur="rename"
      />
    </form>

    <template v-else>
      <span 
        class="todo__title" 
        @dblclick="startEditing"
      >
        {{ todo.title }}
      </span>
      <!-- Remove button appears only on hover -->
      <!-- <button type="button" class="todo__remove" @click="todos.splice(i, 1)"> -->
      <button 
        type="button" 
        class="todo__remove" 
        @click="emit('delete')"
      >
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
