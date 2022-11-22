<script setup lang="ts">
import { remult } from 'remult'
import { ref, onMounted, onUnmounted } from 'vue'
import { Task } from '../shared/Task'
import { TasksController } from '../shared/TasksController';

const tasks = ref<Task[]>([])
const taskRepo = remult.repo(Task)

onMounted(() =>
  onUnmounted(
    taskRepo
      .query({
        where: { completed: undefined }
      })
      .subscribe((applyChanges) => (tasks.value = applyChanges(tasks.value)))
  )
)

const newTaskTitle = ref('')

async function addTask() {
  if (newTaskTitle.value) {
    tasks.value = [
      ...tasks.value,
      await taskRepo.insert({
        title: newTaskTitle.value,
        completed: false,
        id: tasks.value.length + 1
      })
    ]
    newTaskTitle.value = ''
  }
}
async function deleteTask(task: Task) {
  await taskRepo.delete(task)
  tasks.value = tasks.value.filter((t) => t != task)
}

async function saveTask(task: Task) {
  try {
    await taskRepo.save(task)
  } catch (error: any) {
    alert(error.message)
  }
}
async function setAllCompleted(completed: boolean) {
  await TasksController.setAllCompleted(completed);
}
</script>

<template>
  <div>
    <main>
      <input
        v-model="newTaskTitle"
        placeholder="What needs to be done?"
        @keydown.enter="addTask"
      />
      <div v-for="task in tasks">
        <input
          type="checkbox"
          @change="saveTask(task)"
          v-model="task.completed"
        />
        <input v-model="task.title" @blur="saveTask(task)" />
        <button @click="deleteTask(task)">x</button>
      </div>
    </main>
    <div>
      <button @click="setAllCompleted(true)">Set all as completed</button>
      <button @click="setAllCompleted(false)">Set all as uncompleted</button>
    </div>
  </div>
</template>
