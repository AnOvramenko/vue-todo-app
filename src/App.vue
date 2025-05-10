<script setup>
  import { computed, onBeforeMount, ref, watch } from "vue";
  import StatusFilter from './components/StatusFilter.vue';
  import TodoItem from "./components/TodoItem.vue";

  const todos = ref([]);
  const title = ref('');
  const errorMessage = ref('');
  const status = ref('all')

  onBeforeMount(() => {
    try {
      todos.value = JSON.parse(localStorage.getItem('todos'));
    } catch (error) {}

    if (!Array.isArray(todos.value)) {
      todos.value = [];
    }
  })

  const activeTodos = computed(() => {
    return todos.value.filter(todo => !todo.completed)
  });

  const visibleTodos = computed(() => {
    switch (status.value) {
      case 'active':
        return activeTodos.value;
      case 'completed':
        return todos.value.filter(todo => todo.completed);
      default:
        return todos.value;
    }
  })

  watch(
    todos,
    newTodos => {
      localStorage.setItem('todos', JSON.stringify(newTodos));
    },
    {deep: true},
  )

  function addTodo() {
    if (!title.value) {
      errorMessage.value = 'Title should not be empty';

      return;
    };

    todos.value.push({
      id: Date.now(),
      title: title.value,
      completed: false,
    });

    title.value = '';
  }

  function toggleAll() {
    const isCompletedTodos = todos.value.every(todo => todo.completed === true);

    if (isCompletedTodos) {
      todos.value = todos.value.map(todo => ({...todo, completed: !isCompletedTodos}));
    } else {
      todos.value = todos.value.map(todo => {
        if (todo.completed === isCompletedTodos) {
          return ({...todo, completed: !isCompletedTodos})
        }

        return todo;
      })
    }
  }
  
</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <!-- this button should have `active` class only if all todos are
        completed -->
        <button 
          v-if="!!todos.length"
          type="button" 
          class="todoapp__toggle-all"
          :class="{ active: !!activeTodos.length}"
          @click="toggleAll"
        ></button>

        <!-- Add a todo on form submit -->
        <form @submit.prevent="addTodo">
          <input
            type="text"
            class="todoapp__new-todo"
            placeholder="What needs to be done?"
            v-model="title"
            @input="errorMessage = ''"
          />
        </form>
      </header>

      <section class="todoapp__main">
        <!-- This is a completed todo -->
        <!-- <div
          class="todo"
          v-for="(todo, i) of visibleTodos"
          :key="todo.id"
          :class="{ completed: todo.completed }"
        >
          <label class="todo__status-label">
            <input
              type="checkbox"
              class="todo__status"
              v-model="todo.completed"
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
            Remove button appears only on hover
            <button type="button" class="todo__remove" @click="todos.splice(i, 1)">×</button>
          </template>

          overlay will cover the todo while it is being deleted or updated
          <div class="modal overlay" :class="{ 'is-active': false }">
            <div class="modal-background has-background-white-ter"></div>
            <div class="loader"></div>
          </div>
        </div> -->

        <TodoItem 
          v-for="todo of visibleTodos" 
          :key="todo.id" 
          :todo="todo" 
          @delete="todos.splice(todos.indexOf(todo), 1)"
          @update="todos[todos.indexOf(todo)] = $event"
        />
        <!-- This todo is an active todo -->
        
      </section>

      <!-- Hide the footer if there are no todos -->
      <footer class="todoapp__footer" v-if="!!todos.length">
        <span class="todo-count"> {{ activeTodos.length }} items left </span>

        <!-- Active link should have the 'selected' class -->
        <StatusFilter v-model="status"/>
        <!-- :status="status" @change="status = $event"  -->

        <!-- this button should be disabled if there are no completed todos -->
        <button 
          type="button" 
          class="todoapp__clear-completed" 
          :disabled="todos.length === activeTodos.length"
          @click="todos = activeTodos"
        >
          Clear completed
        </button>
      </footer>
    </div>

    <!-- DON'T use conditional rendering to hide the notification -->
    <!-- Add the
    'hidden' class to hide the message smoothly -->
    <div class="notification is-danger is-light has-text-weight-normal" :class="{hidden: !errorMessage}" >
      <button type="button" class="delete" @click="errorMessage = ''"></button>
      <!-- show only one message at a time -->
      {{ errorMessage }}
    </div>
  </div>
</template>


<!-- 
Unable to load todos
<br />
Title should not be empty
<br />
Unable to add a todo
<br />
Unable to delete a todo
<br />
Unable to update a todo 
-->

<!-- v-if="errorMessage"  it is from errorMessage div-->