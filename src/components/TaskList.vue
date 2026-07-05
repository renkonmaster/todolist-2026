<script setup lang="ts">
import { nextTick, ref } from 'vue'

type TodoStatus = 'todo' | 'inProgress' | 'done'

interface Task {
  name: string
  status: TodoStatus
}

const tasks = ref<Task[]>([])
const newTaskName = ref('')
const newTaskStatus = ref<TodoStatus>('todo')
const isOpenAddTask = ref(false)
const isOpenEditTask = ref(false)
const taskErrorMessage = ref('')
const editingTask = ref<Task | null>(null)
const draggingTask = ref<Task | null>(null)
const taskNameInput = ref<HTMLInputElement | null>(null)

const columns = [
  { status: 'todo', label: 'Todo' },
  { status: 'inProgress', label: 'In Progress' },
  { status: 'done', label: 'Done' },
] as const

const tasksByStatus = (status: TodoStatus) => {
  return tasks.value.filter((task) => task.status === status)
}

const openAddTaskModal = async (status: TodoStatus) => {
  newTaskName.value = ''
  newTaskStatus.value = status
  taskErrorMessage.value = ''
  isOpenAddTask.value = true

  await nextTick()
  taskNameInput.value?.focus()
}

const closeAddTaskModal = () => {
  isOpenAddTask.value = false
  taskErrorMessage.value = ''
}

const addTask = () => {
  const name = newTaskName.value.trim()

  if (name === '') {
    taskErrorMessage.value = 'タスク名を入力してください'
    return
  }

  if (tasks.value.some((task) => task.name === name)) {
    taskErrorMessage.value = '同じ名前のタスクは追加できません'
    return
  }

  tasks.value.push({
    name,
    status: newTaskStatus.value,
  })

  newTaskName.value = ''
  newTaskStatus.value = 'todo'
  closeAddTaskModal()
}

const editTask = async (task: Task) => {
  editingTask.value = task
  newTaskName.value = task.name
  newTaskStatus.value = task.status
  taskErrorMessage.value = ''
  isOpenEditTask.value = true

  await nextTick()
  taskNameInput.value?.focus()
}

const closeEditTaskModal = () => {
  isOpenEditTask.value = false
  editingTask.value = null
  taskErrorMessage.value = ''
}

const saveTask = () => {
  if (editingTask.value === null) return

  const name = newTaskName.value.trim()

  if (name === '') {
    taskErrorMessage.value = 'タスク名を入力してください'
    return
  }

  if (tasks.value.some((task) => task.name === name && task !== editingTask.value)) {
    taskErrorMessage.value = '同じ名前のタスクは追加できません'
    return
  }

  editingTask.value.name = name
  editingTask.value.status = newTaskStatus.value

  editingTask.value = null
  closeEditTaskModal()
}

const startDragTask = (task: Task) => {
  draggingTask.value = task
}

const endDragTask = () => {
  draggingTask.value = null
}

const dropTask = (status: TodoStatus) => {
  if (draggingTask.value === null) return

  editingTask.value = draggingTask.value
  newTaskName.value = draggingTask.value.name
  newTaskStatus.value = status
  saveTask()
  draggingTask.value = null
}

const taskCountByStatus = (status: TodoStatus) => {
  return tasks.value.filter((task) => task.status === status).length
}

const statusPercent = (status: TodoStatus) => {
  if (tasks.value.length === 0) return 0

  return Math.round((taskCountByStatus(status) / tasks.value.length) * 100)
}

const progressCircleBackground = (status: TodoStatus) => {
  const percent = statusPercent(status)

  return `conic-gradient(#38bdf8 0 ${percent}%, rgb(255 255 255 / 35%) ${percent}% 100%)`
}
</script>

<template>
  <div class="board">
    <section
      v-for="column in columns"
      :key="column.status"
      class="column"
      @dragover.prevent
      @drop="dropTask(column.status)"
    >
      <div class="column-header">
        <h3>{{ column.label }}</h3>

        <span
          class="progress-circle"
          :style="{ background: progressCircleBackground(column.status) }"
        >
          {{ statusPercent(column.status) }}%
        </span>
      </div>

      <button type="button" class="add-button" @click="openAddTaskModal(column.status)">
        タスクを追加
      </button>

      <div v-for="task in tasksByStatus(column.status)" :key="task.name">
        <div
          class="card"
          draggable="true"
          @click="editTask(task)"
          @dragstart="startDragTask(task)"
          @dragend="endDragTask"
        >
          {{ task.name }}
        </div>
      </div>
    </section>
  </div>

  <div v-if="isOpenAddTask" class="modal-backdrop" @click.self="closeAddTaskModal">
    <form
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="add-task-title"
      @submit.prevent="addTask"
    >
      <h3 id="add-task-title">タスクを追加</h3>

      <label>
        タスク名
        <input ref="taskNameInput" v-model="newTaskName" type="text" />
      </label>

      <label>
        ステータス
        <select v-model="newTaskStatus">
          <option v-for="column in columns" :key="column.status" :value="column.status">
            {{ column.label }}
          </option>
        </select>
      </label>

      <p v-if="taskErrorMessage" class="error-message">{{ taskErrorMessage }}</p>

      <div class="modal-actions">
        <button type="button" @click="closeAddTaskModal">キャンセル</button>
        <button type="submit">追加</button>
      </div>
    </form>
  </div>
  <div v-if="isOpenEditTask" class="modal-backdrop" @click.self="closeEditTaskModal">
    <form
      class="modal"
      role="dialog"
      aria-modal="true"
      aria-labelledby="edit-task-title"
      @submit.prevent="saveTask"
    >
      <h3 id="edit-task-title">タスクを編集</h3>

      <label>
        タスク名
        <input ref="taskNameInput" v-model="newTaskName" type="text" />
      </label>

      <label>
        ステータス
        <select v-model="newTaskStatus">
          <option v-for="column in columns" :key="column.status" :value="column.status">
            {{ column.label }}
          </option>
        </select>
      </label>

      <p v-if="taskErrorMessage" class="error-message">{{ taskErrorMessage }}</p>

      <div class="modal-actions">
        <button type="button" @click="closeEditTaskModal">キャンセル</button>
        <button type="submit">保存</button>
      </div>
    </form>
  </div>
</template>

<style scoped>
:global(body) {
  height: 100dvh;
  margin: 0;
  overflow: hidden;
  color: #182033;
  background:
    linear-gradient(
      115deg,
      rgb(255 255 255 / 0%) 0 34%,
      rgb(255 255 255 / 42%) 34% 35%,
      rgb(255 255 255 / 0%) 35% 100%
    ),
    linear-gradient(
      64deg,
      rgb(88 180 255 / 0%) 0 48%,
      rgb(88 180 255 / 26%) 48% 49%,
      rgb(88 180 255 / 0%) 49% 100%
    ),
    linear-gradient(
      142deg,
      rgb(255 122 162 / 0%) 0 58%,
      rgb(255 122 162 / 24%) 58% 59%,
      rgb(255 122 162 / 0%) 59% 100%
    ),
    repeating-linear-gradient(90deg, rgb(15 23 42 / 5%) 0 1px, transparent 1px 72px),
    repeating-linear-gradient(0deg, rgb(15 23 42 / 4%) 0 1px, transparent 1px 72px),
    linear-gradient(135deg, #dfe9ff 0%, #f8fbff 42%, #e7fff7 100%);
  background-attachment: fixed;
}

:global(body::before) {
  position: fixed;
  inset: 0;
  z-index: -1;
  pointer-events: none;
  content: '';
  background:
    linear-gradient(90deg, rgb(255 255 255 / 34%) 0 1px, transparent 1px 100%),
    linear-gradient(0deg, rgb(255 255 255 / 28%) 0 1px, transparent 1px 100%);
  background-size: 18px 18px;
  mask-image: linear-gradient(to bottom, rgb(0 0 0 / 68%), rgb(0 0 0 / 10%));
}

:global(#app) {
  height: 100dvh;
}

.board {
  display: grid;
  flex: 1;
  grid-template-columns: repeat(3, minmax(220px, 1fr));
  gap: 1.1rem;
  min-height: 0;
  padding: 0;
}

.column-header {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  margin-bottom: 0.75rem;
}

.column-header h3 {
  margin: 0;
}

.progress-circle {
  position: relative;
  display: grid;
  width: 44px;
  height: 44px;
  place-items: center;
  color: #172033;
  font-size: 0.72rem;
  font-weight: 700;
  border-radius: 50%;
}

.progress-circle::before {
  position: absolute;
  inset: 5px;
  content: "";
  background: rgb(255 255 255 / 72%);
  border-radius: 50%;
}

.progress-circle {
  overflow: hidden;
}

.progress-circle {
  isolation: isolate;
}

.progress-circle::before {
  z-index: -1;
}

.column {
  position: relative;
  padding: 0.75rem;
  min-height: 0;
  overflow: auto;
  background:
    linear-gradient(155deg, rgb(255 255 255 / 62%), rgb(255 255 255 / 22%)),
    linear-gradient(35deg, rgb(125 211 252 / 16%), transparent 48%, rgb(244 114 182 / 12%));
  border: 1px solid rgb(255 255 255 / 68%);
  border-radius: 18px;
  box-shadow:
    0 18px 42px rgb(20 37 70 / 14%),
    inset 0 1px 0 rgb(255 255 255 / 86%),
    inset 0 -1px 0 rgb(255 255 255 / 28%);
  backdrop-filter: blur(22px) saturate(165%);
  -webkit-backdrop-filter: blur(22px) saturate(165%);
}

.column::before {
  position: absolute;
  inset: 0;
  pointer-events: none;
  content: '';
  background:
    linear-gradient(120deg, rgb(255 255 255 / 58%), transparent 24%),
    repeating-linear-gradient(90deg, rgb(255 255 255 / 18%) 0 1px, transparent 1px 14px);
  opacity: 0.68;
}

.column > * {
  position: relative;
}

.column h3 {
  margin: 0 0 0.75rem;
  color: #172033;
  font-size: 0.98rem;
  font-weight: 700;
}

.add-button {
  width: 100%;
  padding: 0.55rem 0.75rem;
  margin-bottom: 0.75rem;
  color: #172033;
  cursor: pointer;
  background:
    linear-gradient(180deg, rgb(255 255 255 / 74%), rgb(255 255 255 / 32%)),
    linear-gradient(90deg, rgb(125 211 252 / 18%), rgb(244 114 182 / 14%));
  border: 1px solid rgb(255 255 255 / 78%);
  border-radius: 999px;
  box-shadow:
    0 8px 18px rgb(20 37 70 / 10%),
    inset 0 1px 0 rgb(255 255 255 / 92%);
  backdrop-filter: blur(16px) saturate(160%);
  -webkit-backdrop-filter: blur(16px) saturate(160%);
}

.add-button:hover {
  background:
    linear-gradient(180deg, rgb(255 255 255 / 86%), rgb(255 255 255 / 42%)),
    linear-gradient(90deg, rgb(125 211 252 / 24%), rgb(244 114 182 / 20%));
}

.card {
  padding: 0.75rem;
  margin-top: 0.5rem;
  color: #172033;
  cursor: grab;
  background:
    linear-gradient(150deg, rgb(255 255 255 / 78%), rgb(255 255 255 / 34%)),
    linear-gradient(90deg, rgb(255 255 255 / 32%), transparent);
  border: 1px solid rgb(255 255 255 / 78%);
  border-radius: 14px;
  box-shadow:
    0 12px 26px rgb(20 37 70 / 12%),
    inset 0 1px 0 rgb(255 255 255 / 92%),
    inset 0 -1px 0 rgb(255 255 255 / 34%);
  backdrop-filter: blur(18px) saturate(165%);
  -webkit-backdrop-filter: blur(18px) saturate(165%);
}

.card:hover {
  transform: translateY(-1px);
  box-shadow:
    0 16px 34px rgb(20 37 70 / 16%),
    inset 0 1px 0 rgb(255 255 255 / 94%),
    inset 0 -1px 0 rgb(255 255 255 / 38%);
}

.card:active {
  cursor: grabbing;
}

.modal-backdrop {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  padding: 1rem;
  background:
    linear-gradient(135deg, rgb(15 23 42 / 22%), rgb(15 23 42 / 8%)),
    repeating-linear-gradient(90deg, rgb(255 255 255 / 9%) 0 1px, transparent 1px 38px);
  backdrop-filter: blur(14px) saturate(130%);
  -webkit-backdrop-filter: blur(14px) saturate(130%);
}

.modal {
  display: grid;
  width: min(100%, 420px);
  gap: 0.75rem;
  padding: 1rem;
  background:
    linear-gradient(145deg, rgb(255 255 255 / 78%), rgb(255 255 255 / 36%)),
    linear-gradient(35deg, rgb(125 211 252 / 16%), transparent 48%, rgb(244 114 182 / 14%));
  border: 1px solid rgb(255 255 255 / 82%);
  border-radius: 22px;
  box-shadow:
    0 26px 76px rgb(15 23 42 / 26%),
    inset 0 1px 0 rgb(255 255 255 / 92%),
    inset 0 -1px 0 rgb(255 255 255 / 32%);
  backdrop-filter: blur(28px) saturate(170%);
  -webkit-backdrop-filter: blur(28px) saturate(170%);
}

.modal h3 {
  margin: 0;
  color: #172033;
}

.modal label {
  display: grid;
  gap: 0.25rem;
  color: #334155;
}

.modal input,
.modal select {
  padding: 0.5rem;
  color: #172033;
  background: rgb(255 255 255 / 48%);
  border: 1px solid rgb(255 255 255 / 82%);
  border-radius: 12px;
  box-shadow:
    inset 0 1px 0 rgb(255 255 255 / 84%),
    0 8px 18px rgb(20 37 70 / 8%);
}

.modal input:focus,
.modal select:focus {
  outline: 2px solid rgb(88 180 255 / 42%);
  outline-offset: 2px;
}

.error-message {
  margin: 0;
  color: #cf222e;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
}

.modal-actions button {
  padding: 0.5rem 0.75rem;
  color: #172033;
  cursor: pointer;
  background: rgb(255 255 255 / 48%);
  border: 1px solid rgb(255 255 255 / 78%);
  border-radius: 999px;
  box-shadow:
    0 8px 18px rgb(20 37 70 / 10%),
    inset 0 1px 0 rgb(255 255 255 / 88%);
}
</style>
