<script setup lang="ts">
import ClickCounter from './components/ClickCounter.vue'
import ItemList from './components/ItemList.vue'
import TaskList from './components/TaskList.vue'
import TodoList from './components/TodoList.vue'
import WelcomeMessage from './components/WelcomeMessage.vue'
</script>

<template>
  <main>
    <WelcomeMessage msg="Webエンジニアになろう講習会へようこそ" />
    <ClickCounter />
    <ItemList />
    <TodoList />
    <TaskList />
  </main>
</template>

<style scoped>
main {
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  height: 100dvh;
  overflow: hidden;
  padding: 2rem;
}
</style>
