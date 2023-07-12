<template>
  <div>
    <h4 v-if="!isLoggedIn" class="login-text">Please Login</h4>
    <div v-else class="task-container">
      <div class="task-header">
        <button class="btn btn-primary" @click="openForm">Add Task</button>
      </div>

      <div class="task-list">
        <div>
          <h5>To Do List:</h5>
        </div>

        <div class="task-form-container" v-if="showForm">
          <form @submit="submitTask" class="task-form">
            <div class="form-group m-3">
              <label for="taskName">Task Name</label>
              <input
                type="text"
                class="form-control"
                id="taskName"
                v-model.trim="name"
                :class="{ 'is-invalid': !isNameValid && name.length > 0 }"
                required
              />
              <p v-if="!isNameValid && name.length > 0" class="error-message">
                Name must be characters between 2 and 20 characters
              </p>
            </div>
            <div class="form-group m-3">
              <label for="dueDate">Due Date</label>
              <input
                type="date"
                class="form-control"
                id="dueDate"
                v-model="dueDate"
                :class="{ 'is-invalid': !isDueDateValid && dueDate.length > 0 }"
                required
              />
              <p
                v-if="!isDueDateValid && dueDate.length > 0"
                class="error-message"
              >
                Please select a due date.
              </p>
            </div>
            <div class="form-group m-3">
              <label for="description">Description</label>
              <textarea
                class="form-control"
                id="description"
                v-model.trim="description"
                :class="{
                  'is-invalid': !isDescriptionValid && description.length > 0,
                }"
              ></textarea>
              <p
                v-if="!isDescriptionValid && description.length > 0"
                class="error-message"
              >
                Description must be between 2 and 30 characters.
              </p>
            </div>
            <button
              type="submit"
              class="btn btn-primary m-3"
              :disabled="!isFormValid"
            >
              {{ editMode ? "Update Task" : "Add Task" }}
            </button>
          </form>
        </div>

        <div class="task-cards-container row">
          <div class="card m-1" v-for="task in tasks" :key="task.id">
            <div class="card-body">
              <div class="task-header row">
                <div class="col">
                  <h5 class="card-title">{{ task.name }}</h5>
                </div>
                <div class="col-auto task-actions">
                  <button
                    class="btn btn-secondary btn-sm"
                    @click="editTask(task.id)"
                  >
                    <i class="fas fa-edit"></i>
                  </button>
                  <button
                    class="btn btn-danger btn-sm"
                    @click="deleteTask(task.id)"
                  >
                    <i class="fas fa-trash"></i>
                  </button>
                  <button
                    class="btn btn-success btn-sm"
                    @click="markAsDone(task)"
                  >
                    Mark as Done
                  </button>
                </div>
              </div>
              <hr />
              <div>
                <h6 class="card-text">{{ task.description }}</h6>
              </div>
              <div>
                <p class="card-subtitle mb-2 text-muted">
                  Due Date: {{ task.dueDate }} |
                  <span class="days-left">{{ daysLeft(task) }} days left</span>
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-if="isLoggedIn" class="completed-task">
    <CompletedTask />
  </div>
</template>

<script setup lang="ts">
import { useStore } from "vuex";
import { ref, provide, computed } from "vue";
import CompletedTask from "@/views/CompletedTask.vue";
import "@/styles/MyTask.css";
import {
  name,
  dueDate,
  description,
  isNameValid,
  isDueDateValid,
  isDescriptionValid,
  isFormValid,
} from "@/mixins/FormValidations.ts";

const store = useStore();
const tasks = ref([]);
const taskId = ref(1);
const showForm = ref(false);
const editMode = ref(false);
const editIndex = ref(-1);
const completedTasks = ref([]);

provide("completedTasks", completedTasks);

const isLoggedIn = computed(() => store.getters.userAuthCheck);

const openForm = () => {
  showForm.value = true;
  editMode.value = false;
  resetForm();
};

const submitTask = (event) => {
  event.preventDefault();

  if (editMode.value) {
    updateTask();
  } else {
    addTask();
  }

  resetForm();
  showForm.value = false;
};

const addTask = () => {
  const task = {
    id: taskId.value++,
    name: name.value,
    dueDate: dueDate.value,
    description: description.value,
  };
  tasks.value.push(task);
};

const updateTask = () => {
  const task = tasks.value[editIndex.value];
  task.name = name.value;
  task.dueDate = dueDate.value;
  task.description = description.value;
};
const daysLeft = (task) => {
  const dueDate = new Date(task.dueDate);
  const currentDate = new Date();
  const timeDifference = dueDate.getTime() - currentDate.getTime();
  const daysDifference = Math.ceil(timeDifference / (1000 * 3600 * 24));
  return daysDifference;
};

const markAsDone = (task) => {
  deleteTask(task.id);
  completedTasks.value.push(task);
};

const deleteTask = (taskId) => {
  const taskIndex = tasks.value.findIndex((task) => task.id === taskId);
  if (taskIndex > -1) {
    tasks.value.splice(taskIndex, 1);
  }
};

const editTask = (taskId) => {
  const taskIndex = tasks.value.findIndex((task) => task.id === taskId);
  if (taskIndex > -1) {
    const task = tasks.value[taskIndex];
    name.value = task.name;
    dueDate.value = task.dueDate;
    description.value = task.description;
    editMode.value = true;
    editIndex.value = taskIndex;
    showForm.value = true;
  }
};
const resetForm = () => {
  name.value = "";
  dueDate.value = "";
  description.value = "";
};
</script>

<style scoped>
.is-invalid {
  border-color: red !important;
}

.error-message {
  color: red;
  font-size: 12px;
  margin-top: 4px;
}
</style>