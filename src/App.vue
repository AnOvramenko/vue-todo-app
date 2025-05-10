<script setup>
  import { computed, nextTick, onMounted, ref, watch } from "vue";
  import StatusFilter from './components/StatusFilter.vue';
  import TodoItem from "./components/TodoItem.vue";
  import ErrorMessage from "./components/ErrorMessage.vue";
  import * as todoApi from './api/todos';

  const todos = ref([]);
  const title = ref('');
  const errorMessage = ref(null);
  const status = ref('all')
  const addTodoField = ref(null);
  const tempTodo = ref(null);

  onMounted(async () => {
   try {
      todos.value = await todoApi.getTodos();
   } catch (error) {
      errorMessage.value.show('Unable to load todos');
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
    async () => {
    await nextTick();
    addTodoField.value.focus();
  });

  const setLoading = (id, status = true) => {
    todos.value = todos.value.map(todo => {
      if (todo.id === id) {
        return {...todo, loading: status};
      }

      return todo;
    })
  }

  async function addTodo() {
    if (!title.value) {
      errorMessage.value.show('Title should not be empty');

      return;
    };

    tempTodo.value = {id: 0, title: title.value, completed: false, loading: true};

    try {
      const newTodo = await todoApi.createTodo(title.value);

      todos.value.push(newTodo);
      title.value = '';
    } catch (error) {
      errorMessage.value.show('Unable to add a todo');
    } finally {
      tempTodo.value = null;
    }
  }

  const removeTodo = async (todoId) => {
    setLoading(todoId);

    try {
      await todoApi.deleteTodo(todoId);
      todos.value = todos.value.filter(todo => todo.id !== todoId);
    } catch (error) {
      errorMessage.value.show('Unable to delete a todo');
    } finally {
      setLoading(todoId, false);
    }
  };

  const updateTodo = async ({id, title, completed}) => {
    setLoading(id);

    try {
      const updatedTodo = await todoApi.updateTodo({id, title, completed});
      const currentTodo = todos.value.find(todo => todo.id === id);
  
      Object.assign(currentTodo, updatedTodo);
    } catch (error) {
      errorMessage.value.show('Unable to update a todo');
    } finally {
      setLoading(id, false);
    }
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

    completedTodos.forEach(({id}) => removeTodo(id));
  }

</script>

<template>
  <div class="todoapp">
    <h1 class="todoapp__title">todos</h1>

    <div class="todoapp__content">
      <header class="todoapp__header">
        <button 
          v-if="!!todos.length"
          type="button" 
          class="todoapp__toggle-all"
          :class="{ active: !!activeTodos.length}"
          @click="toggleAll"
        ></button>

        <form @submit.prevent="addTodo">
          <input
            ref="addTodoField"
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
        >


        <TodoItem 
          v-for="todo of visibleTodos" 
          :key="todo.id" 
          :todo="todo" 
          @delete="removeTodo(todo.id)"
          @update="updateTodo($event)"
        />

        <TodoItem :todo="tempTodo" v-if="!!tempTodo"/>
      </TransitionGroup>

      <footer class="todoapp__footer" v-if="!!todos.length">
        <span class="todo-count"> {{ activeTodos.length }} items left </span>

        <StatusFilter v-model="status"/>

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

    <ErrorMessage @close="errorMessage = ''" class="is-warning" ref="errorMessage" />
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
