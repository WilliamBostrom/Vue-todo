<!-- App.vue -->
<script setup lang="ts">
import { ref, onMounted } from "vue";
import HelloWorld from "./components/TodoList.vue";

// Types
interface Todo {
  id: string;
  title: string;
  completed: boolean;
}

// Reactive references
let todos = ref<Todo[]>([]);
const error = ref<string | null>(null);
const isLoading = ref(false);
const isAdding = ref(false);
const scrollOnAdd = ref("top");
// const disabledItems = ref<string[]>([]);

// Mounting
onMounted(() => {
  loadTodos();
});

// Load todos
async function loadTodos() {
  isLoading.value = true;
  try {
    const response = await fetch(
      "https://jsonplaceholder.typicode.com/todos?_limit=10"
    );
    if (response.ok) {
      todos.value = await response.json();
    } else {
      error.value = "An error has occurred";
    }
  } catch (err) {
    error.value = "Network error";
  } finally {
    isLoading.value = false;
  }
}

// Add todo
async function handleAddTodo(title: string) {
  isAdding.value = true;
  try {
    const response = await fetch("https://jsonplaceholder.typicode.com/todos", {
      method: "POST",
      body: JSON.stringify({
        title,
        completed: false,
      }),
      headers: {
        "Content-type": "application/json; charset=UTF-8",
      },
    });

    if (response.ok) {
      const todo = await response.json();
      todos.value = [{ ...todo, id: crypto.randomUUID() }, ...todos.value];
      console.log(todo);
    } else {
      alert("An error has occurred.");
    }
  } catch (err) {
    alert("Network error");
  } finally {
    isAdding.value = false;
  }
}

// Remove todo
async function handleRemoveTodo(id: string) {
  // if (disabledItems.value.includes(id)) return;

  // disabledItems.value.push(id);
  try {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${id}`,
      {
        method: "DELETE",
      }
    );

    if (response.ok) {
      todos.value = todos.value.filter((t) => t.id !== id);
    } else {
      alert("An error has occurred.");
    }
  } catch (err) {
    alert("Network error");
  }
  // finally {
  //   disabledItems.value = disabledItems.value.filter((itemId) => itemId !== id);
  // }
}

// Toggle todo

async function handleToggleTodo(payload: { id: number; value: boolean }) {
  const { id, value } = payload;

  try {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${id}`,
      {
        method: "PATCH",
        body: JSON.stringify({ completed: value }),
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      }
    );

    if (response.ok) {
      const updatedTodo = await response.json();
      todos.value = todos.value.map((todo) =>
        Number(todo.id) === id
          ? { ...todo, completed: updatedTodo.completed }
          : todo
      );
      console.log(updatedTodo);
    } else {
      alert("An error has occurred.");
    }
  } catch (error) {
    console.error("Error toggling todo:", error);
    alert("Network error");
  }
}
</script>

<template>
  <div>
    <a href="https://vitejs.dev" target="_blank">
      <img src="/vite.svg" class="logo" alt="Vite logo" />
    </a>
    <a href="https://vuejs.org/" target="_blank">
      <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
    </a>
  </div>
  <HelloWorld
    :todos="todos"
    :error="error"
    :is-loading="isLoading"
    :disable-adding="isAdding"
    :scroll-on-add="scrollOnAdd"
    @addtodo="handleAddTodo"
    @removetodo="handleRemoveTodo"
    @toggletodo="handleToggleTodo"
  />
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
