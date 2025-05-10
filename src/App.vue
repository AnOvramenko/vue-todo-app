<script setup>
  import { computed, onBeforeMount, onMounted, ref, watch } from "vue";
  import StatusFilter from './components/StatusFilter.vue';
  import TodoItem from "./components/TodoItem.vue";
  import * as todoApi from './api/todos';

  const todos = ref([]);
  const title = ref('');
  const errorMessage = ref('');
  const status = ref('all')

  // onBeforeMount(() => {
  //   try {
  //     todos.value = JSON.parse(localStorage.getItem('todos'));
  //   } catch (error) {}

  //   if (!Array.isArray(todos.value)) {
  //     todos.value = [];
  //   }
  // })

  onMounted(async () => {
    todos.value = await todoApi.getTodos();
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

  // watch(
  //   todos,
  //   newTodos => {
  //     localStorage.setItem('todos', JSON.stringify(newTodos));
  //   },
  //   {deep: true},
  // )

  async function addTodo() {
    if (!title.value) {
      errorMessage.value = 'Title should not be empty';

      return;
    };

    const newTodo = await todoApi.createTodo(title.value);

    todos.value.push(newTodo);

    // todos.value.push({
    //   id: Date.now(),
    //   title: title.value,
    //   completed: false,
    // });

    title.value = '';
  }

  const deleteTodo = async (todoId) => {
    await todoApi.deleteTodo(todoId);
    todos.value = todos.value.filter(todo => todo.id !== todoId);
  };

  const updateTodo = async ({id, title, completed}) => {
    const updatedTodo = await todoApi.updateTodo({id, title, completed});
    const currentTodo = todos.value.find(todo => todo.id === id);

    Object.assign(currentTodo, updatedTodo);
  }

  function toggleAll() {
    const unCompletedTodos = todos.value.filter(todo => !todo.completed);

    if (unCompletedTodos.length === 0 || unCompletedTodos.length === todos.value.length) {
      todos.value.forEach(todo => updateTodo({...todo, completed: !todo.completed}));
    } else {
      unCompletedTodos.forEach(todo => updateTodo({...todo, completed: true}));
    }
  }

  const clearAllCompleted = () => {
    const completedTodos = todos.value.filter(todo => todo.completed);

    completedTodos.forEach(({id}) => todoApi.deleteTodo(id));
    todos.value = activeTodos.value;
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

        <TransitionGroup 
          tag="section"
          name="todolist"
          class="todoapp__main"
          v-if="!!todos.length"
        >

        <TodoItem 
          v-for="todo of visibleTodos" 
          :key="todo.id" 
          :todo="todo" 
          @delete="deleteTodo(todo.id)"
          @update="updateTodo($event)"
        />
      </TransitionGroup>

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
          @click="clearAllCompleted"
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

<style scoped>
  .todolist-enter-active,
  .todolist-leave-active {
    max-height: 60px;
    transition: all 0.5s ease;
  }
  .todolist-enter-from,
  .todolist-leave-to {
    opacity: 0;
    max-height: 0;
    transform: scaleY(0);
  }
</style>

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