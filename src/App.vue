<script setup>
import { ref, onMounted, computed, watch } from 'vue';

const todos = ref([]);
const name = ref('');
const todoCount = ref(0);
const inputContent = ref('');
const inputCategory = ref('');

// 排序，最新的在最上面
const todosAsc = computed(() =>
  todos.value.sort((a, b) => {
    return b.createAt - a.createAt;
  })
);

// 創建Todo物件
const addTodo = () => {
  if (inputContent.value.trim() === '' || inputCategory.value === null) return;
  todos.value.push({
    content: inputContent.value,
    category: inputCategory.value,
    createAt: new Date().getTime(),
    isDone: false,
  });
  todoCount.value++;
  inputContent.value = '';
  inputCategory.value = null;
};

// 刪除單筆
const removeTodo = (todo) => {
  const index = todos.value.indexOf(todo);
  todos.value.splice(index, 1);
  todoCount.value--;
};

// 刪除已完成（可多筆）
const deleteCompleted = () => {
  for (let i = todos.value.length - 1; i >= 0; i--) {
    const item = todos.value[i];
    if (item.isDone) {
      todos.value.splice(i, 1);
      todoCount.value--;
    }
  }
};

// 監聽todos、name，並存入localStorage
watch(
  todos,
  (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem('name', newVal);
});


onMounted(() => {
  name.value = localStorage.getItem('name') || '';
  todos.value = JSON.parse(localStorage.getItem('todos')) || [];
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        Hi, <input type="text" placeholder="Type your name" v-model="name" />
      </h2>
    </section>
    <section class="create-todo">
      <h3>CREATE A TODO</h3>
      <form @submit.prevent="addTodo">
        <h4>What's on your todo list</h4>
        <input type="text" placeholder="" v-model="inputContent" />
        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="business"
              v-model="inputCategory"
            />
            <span class="bubble business"> </span>
            <div>Business</div>
          </label>
          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="inputCategory"
            />
            <span class="bubble personal"> </span>
            <div>Personal</div>
          </label>
        </div>
        <input type="submit" value="Add todo" />
      </form>
    </section>
    <section class="todo-list">
      <div class="todo-list-wrap">
        <h3>TODO LIST</h3>
        <div class="delete-completed">
          <button class="delete-completed-btn" @click="deleteCompleted()">
            Delete completed items
          </button>
        </div>
      </div>
      <div class="list">
        <div
          v-for="todo in todosAsc"
          :class="`todo-item ${todo.isDone && isDone}`"
        >
          <label>
            <input type="checkbox" v-model="todo.isDone" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>
          <div class="createAt">
            {{ new Date(todo.createAt).toLocaleString() }}
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
      <div class="todo-list-info">
        <div class="todo-count">
          <span>{{ todoCount }}</span> Items left
        </div>      
      </div>
    </section>
  </main>
</template>
