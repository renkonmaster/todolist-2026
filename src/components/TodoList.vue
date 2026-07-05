<script setup lang="ts">
import { computed, ref } from 'vue'

interface Todo {
  id: number
  name: string
  done: boolean
}

const todos = ref<Todo[]>([])
const newTodoName = ref('')
const newTodoDone = ref(false)
const nextTodoId = ref(1)

const sortedTodos = computed(() => [
  ...todos.value.filter((todo) => !todo.done),
  ...todos.value.filter((todo) => todo.done),
])

const todoCountMessage = computed(
  () => `未完了: ${todos.value.filter((todo) => !todo.done).length} / 全体: ${todos.value.length}`,
)

const addTodo = () => {
  const name = newTodoName.value.trim()

  if (name === '') return
  if (todos.value.some((todo) => todo.name === name)) return

  todos.value.push({
    id: nextTodoId.value,
    name,
    done: newTodoDone.value,
  })

  nextTodoId.value++
  newTodoName.value = ''
  newTodoDone.value = false
}

const toggleTodo = (id: number) => {
  const todo = todos.value.find((todo) => todo.id === id)

  if (todo === undefined) return

  todo.done = !todo.done
}

const deleteTodo = (id: number) => {
  todos.value = todos.value.filter((todo) => todo.id !== id)
}

const statusLabel = (done: boolean) => (done ? '完了' : '未完了')
</script>

<template>
  <section class="todo-list">
    <h2>TodoList</h2>

    <p>{{ todoCountMessage }}</p>

    <ul>
      <li v-for="todo in sortedTodos" :key="todo.id" :class="{ done: todo.done }">
        <span>名前: {{ todo.name }}</span>
        <span>状態: {{ statusLabel(todo.done) }}</span>
        <button type="button" @click="toggleTodo(todo.id)">切り替え</button>
        <button type="button" @click="deleteTodo(todo.id)">削除</button>
      </li>
    </ul>

    <form class="todo-form" @submit.prevent="addTodo">
      <label>
        名前
        <input v-model="newTodoName" type="text" />
      </label>
      <label>
        完了済み
        <input v-model="newTodoDone" type="checkbox" />
      </label>
      <button type="submit">追加</button>
    </form>
  </section>
</template>

<style scoped>
.todo-list {
  margin-top: 1rem;
}

.todo-form {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

li {
  margin-bottom: 0.5rem;
}

li span,
li button {
  margin-right: 0.5rem;
}

.done {
  color: gray;
  text-decoration: line-through;
}
</style>
