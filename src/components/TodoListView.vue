<script setup lang="ts">
import {ref, reactive, onMounted, computed} from 'vue'
import axios from "axios"
const data: any = reactive({ todos: [] });

const description = ref("");

async function addItem() {
    if (!description.value) return;
    if (data.todos.some((item: any) => item.description === description.value)) return;
    if (data.todos.filter((item: any) => !item.done).length > 4) return;
    const item = {id: Math.random().toString(36).slice(2, 7), description: description.value, done: false }
    data.todos.push(item)
    await axios.post("http://localhost:3000/todos", item)
}

async function removeItem(item: any) {
    data.todos.splice(data.todos.indexOf(item), 1)
    await axios.delete(`http://localhost:3000/todos/${item.id}`)
}

async function toogleDone(item: any) {
    item.done = !item.done
    await axios.put(`http://localhost:3000/todos/${item.id}`, item)
}
const completed = computed(() => {
  const total = data.todos.length
  const done = data.todos.filter((item: any) => item.done).length
  return Math.round((done/total) * 10)
})

onMounted(async () => {
    const response = await axios.get("http://localhost:3000/todos")
    data.todos = response.data;
})
</script>

<template>
    <div v-if="data.todos.length === 0">
        No items
    </div>
    <span class=completed">{{completed}}%</span>
    <div v-for="item in data.todos">
        <span :style="{ 'text-decoration': (item.done) ? 'line-through' : '' }">{{ item.description }}</span> {{ item.done }}
        <button @click="removeItem(item)">Remove</button>
        <button @click="toogleDone(item)">Done/undone</button>
    </div>
    <input type="text" v-model="description" @keyup.enter="addItem()" />
</template>

<style scoped>

</style>
