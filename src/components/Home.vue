<template>
  <div class="home">
    <div class="flex mb-6 border-b border-gray-400 pb-6">
      <div class="flex">
        <h2 class="font-bold text-3xl">TaskList</h2>
        <div class="flex justify-center items-center ml-4">
          <moon-loader
            :loading="loading"
            :color="color"
            :size="size"
          ></moon-loader>
        </div>
      </div>
      <button
        @click="newTask"
        class="ml-auto bg-blue-500 py-2 px-4 rounded-md text-white hover:bg-blue-400 "
      >
        Nova Tarefa
      </button>
    </div>
    <div v-if="isAddingTask" class="p-4 rounded flex ">
      <div class="ml-2">
        <input type="checkbox" v-model="newTaskDone" />
      </div>
      <div class="flex-1 pl-4">
        <div class="mb-2">
          <input
            ref="teste"
            type="text"
            placeholder="Título"
            v-model="newTaskTitle"
            class="p-2 w-full border border-gray-400 rounded-sm"
          />
        </div>
        <div>
          <textarea
            rows="2"
            placeholder="Descrição"
            v-model="newTaskDescription"
            class="p-2 w-full border border-gray-400 rounded-sm"
          ></textarea>
        </div>
      </div>
      <div class="flex flex-col w-32 ml-4">
        <button
          @click="saveNewTask"
          class="border border-gray-600 p-2 rounded text-gray-600 mb-2 hover:border-blue-600 hover:text-blue-600 "
        >
          Adicionar
        </button>
        <button
          @click="cancelNewTask"
          class="border border-gray-500 p-2 rounded text-gray-500 hover:border-gray-600 hover:text-gray-600"
        >
          Cancelar
        </button>
      </div>
    </div>

    <div v-if="isEditingTask" class="p-4 rounded flex ">
      <div class="ml-2">
        <input type="checkbox" v-model="newTaskDone" />
      </div>
      <div class="flex-1 pl-4">
        <div class="mb-2">
          <input
            ref="teste"
            type="text"
            placeholder="Título"
            v-model="newTaskTitle"
            class="p-2 w-full border border-gray-400 rounded-sm"
          />
        </div>
        <div>
          <textarea
            rows="2"
            placeholder="Descrição"
            v-model="newTaskDescription"
            class="p-2 w-full border border-gray-400 rounded-sm"
          ></textarea>
        </div>
      </div>
      <div class="flex flex-col w-32 ml-4">
        <button
          @click="updateTask"
          class="border border-gray-600 p-2 rounded text-gray-600 mb-2 hover:border-blue-600 hover:text-blue-600 "
        >
          Atualizar
        </button>
        <button
          @click="cancelNewTask"
          class="border border-gray-500 p-2 rounded text-gray-500 hover:border-gray-600 hover:text-gray-600"
        >
          Cancelar
        </button>
      </div>
    </div>
    <div
      v-for="task in tasks"
      :key="task.id"
      class="p-4 rounded flex border rounded my-2 border-gray-400"
      @dblclick="editTask(task)"
    >
      <div class="ml-2">
        <input type="checkbox" v-model="task.done" @click="toggleTask(task)" />
      </div>
      <div class="pl-4 flex-1" :class="{ 'text-gray-600': task.done }">
        <h3
          class="font-bold"
          :class="[
            { 'text-gray-600': task.done },
            { 'line-through': task.done }
          ]"
        >
          {{ task.title }}
        </h3>
        <p>{{ task.description }}</p>
      </div>
      <div class="task-actions">
        <button
          @click="deleteTask(task)"
          class="text-gray-600 hover:text-gray-800"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 20 20"
            fill="currentColor"
            class="w-4 h-4"
          >
            <path
              fill-rule="evenodd"
              d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
              clip-rule="evenodd"
            />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import MoonLoader from "vue-spinner/src/MoonLoader.vue";
const BASE_URL = "http://159.65.218.85:8080"; //process.env.VUE_APP_BASEURL;

export default {
  name: "Home",
  components: {
    MoonLoader
  },
  data() {
    return {
      loading: true,
      color: "#4299e1",
      size: "40px",
      tasks: [],
      isAddingTask: false,
      isEditingTask: false,
      editingTaskId: 0,
      newTaskDone: false,
      newTaskTitle: "",
      newTaskDescription: ""
    };
  },
  mounted() {
    this.loadTasks();
  },
  methods: {
    toggleTask(task) {
      console.log("toggle: " + task.id);
      const patch = {
        done: task.done
      };

      axios.patch(BASE_URL + "/api/v1/tasks/" + task.id, patch);
    },
    loadTasks() {
      this.loading = true;
      axios.get(BASE_URL + "/api/v1/tasks").then(result => {
        this.tasks = result.data;
        this.loading = false;
      });
    },
    cancelNewTask() {
      this.newTaskTitle = "";
      this.newTaskDescription = "";
      this.newTaskDone = false;
      this.isAddingTask = false;
      this.isEditingTask = false;
      this.editingTaskId = 0;
    },
    newTask() {
      this.isAddingTask = true;

      this.$nextTick(() => {
        this.$refs.teste.focus();
      });
    },
    editTask(task) {
      this.editingTaskId = task.id;
      this.newTaskTitle = task.title;
      this.newTaskDescription = task.description;
      this.newTaskDone = task.done;
      this.isEditingTask = true;

      this.$nextTick(() => {
        this.$refs.teste.focus();
      });
    },
    async updateTask() {
      const updatedTask = {
        title: this.newTaskTitle,
        description: this.newTaskDescription,
        done: this.newTaskDone
      };

      await axios.put(
        BASE_URL + "/api/v1/tasks/" + this.editingTaskId,
        updatedTask
      );

      this.loadTasks();

      this.cancelNewTask();
    },
    async deleteTask(task) {
      await axios.delete(BASE_URL + "/api/v1/tasks/" + task.id);
      this.loadTasks();
    },
    async saveNewTask() {
      const newTask = {
        title: this.newTaskTitle,
        description: this.newTaskDescription,
        done: this.newTaskDone
      };

      await axios.post(BASE_URL + "/api/v1/tasks", newTask);

      this.loadTasks();

      this.cancelNewTask();
    }
  }
};
</script>

<style scoped>
.task {
  padding: 10px;
  border: 1px solid gray;
  border-radius: 10px;
  margin: 10px 0px;
  display: flex;
}

.task .check-area {
  padding-right: 10px;
}

.task .task-info {
  display: flex;
  flex-direction: column;
  flex: 1;
}
</style>
