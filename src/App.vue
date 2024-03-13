<template>
  <div id="app">
<img src="@/assets/todolist.png" width="100" alt="icon">    
<h1>To-Do List</h1>
<hr>
    <div>
      <h2>Active Tasks</h2>
      <ul>
        <li v-for="task in openTasks" :key="task.id">
          <span class="task-text">{{ task.text }}</span> 
          (Created on {{ formatDate(task.createdAt) }})
          <button class="complete-button" @click="completeTask(task.id)">Complete</button>
          <button class="delete-button" @click="deleteTask(task.id)">Delete</button>
        </li>
      </ul>
    </div>
<hr>
    <div id="completed-tasks-section">
      <h2>Completed Tasks</h2>
      <ul>
        <li v-for="task in completedTasks" :key="task.id">
          <span class="task-text">{{ task.text }}</span> 
          (Completed on {{ formatDate(task.completedAt) }})
          <button class="delete-button" @click="deleteTask(task.id)">Delete</button>
        </li>
      </ul>
    </div>
<hr>
    <div id="add-task-section">
      <h2>Add New Task</h2>
      <input v-model="newTask" />
      <button @click="addTask">Add Task</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";

export default {
  data() {
    return {
      tasks: [],
      newTask: ""
    };
  },
  computed: {
    openTasks() {
      return this.tasks.filter(task => !task.completed);
    },
    completedTasks() {
      return this.tasks.filter(task => task.completed);
    }
  },
  methods: {
    async deleteTask(taskId) {
      try {
        await axios.delete(`http://localhost:3000/tasks/${taskId}`);
        this.tasks = this.tasks.filter(task => task.id !== taskId);
      } catch (error) {
        console.error("Error deleting task", error);
      }
    },
    async fetchTasks() {
      try {
        const response = await axios.get("http://localhost:3000/tasks");
        this.tasks = response.data;
      } catch (error) {
        console.error("Error fetching tasks", error);
      }
    },
    async addTask() {
      if (this.newTask.trim() === "") return;

      try {
        const response = await axios.post("http://localhost:3000/tasks", {
          text: this.newTask,
          createdAt: moment().format(),
          completed: false
        });
        this.tasks.push(response.data);
        this.newTask = "";
      } catch (error) {
        console.error("Error adding task", error);
      }
    },
    async completeTask(taskId) {
      try {
        const response = await axios.patch(
          `http://localhost:3000/tasks/${taskId}`,
          {
            completed: true,
            completedAt: moment().format()
          }
        );

        const updatedTask = response.data;
        const index = this.tasks.findIndex(task => task.id === updatedTask.id);

        // Directly update the array using assignment
        this.tasks[index] = updatedTask;
      } catch (error) {
        console.error("Error completing task", error);
      }
    },
    formatDate(dateString) {
      return moment(dateString).format("MMMM D, YYYY [at] h:mm A");
    }
  },
  created() {
    this.fetchTasks();
  }
};
</script>
<style>
#app {
  text-align: center;
  margin-top: 2rem;
  background-color: #f0f0f0; /* Light gray background */
  padding: 1rem;
  border-radius: 8px;
}

h1 {
  font-size: 2rem;
  color: #333;
}

ul {
  list-style-type: square;
  padding: 0;
}

li {
  margin: 1rem 0;
  padding: 1rem;
  border: 0.5px solid #ccc;
  border-radius: 2px;
  background-color: #fff;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1.2rem; /* Increased font size */
  transition: background-color 0.3s ease, transform 0.3s ease;
}

li:hover {
  background-color: #e0e0e0;
  transform: scale(1.05);
}

.task-text {
  flex-grow: 1;
}

.completed-task {
  text-decoration: line-through;
  color: #777;
}

li button {
  cursor: pointer;
  padding: 0.5rem;
  border: none;
  border-radius: 4px;
  color: #fff;
}

.complete-button,
.delete-button {
  margin-left: 10px; /* Adjust this value to bring buttons closer */
}

.complete-button {
  background-color: #4caf50; /* Green */
  transition: background-color 0.3s ease;
}

.complete-button:hover {
  background-color: #45a049; /* Darker green on hover */
}

.delete-button {
  background-color: #f44336; /* Red */
  transition: background-color 0.3s ease;
}

.delete-button:hover {
  background-color: #d32f2f; /* Darker red on hover */
}

#add-task-section {
  margin-top: 1rem;
}

#add-task-section input {
  padding: 0.5rem;
  margin-right: 0.5rem;
}

#add-task-section button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #2196f3; /* Blue */
  color: #fff;
  cursor: pointer;
}

#completed-tasks-section {
  margin-top: 1rem;
}

#completed-tasks-section li {
  color: #777;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background-color 0.3s ease;
}

#completed-tasks-section li button {
  cursor: pointer;
  padding: 0.2rem;
  border: none;
  border-radius: 4px;
  color: #fff;
}

#completed-tasks-section li .delete-button {
  margin-left: 10px; /* Adjust this value to bring buttons closer */
  background-color: #f44336; /* Red */
  transition: background-color 0.3s ease;
}

#completed-tasks-section li:hover {
  background-color: #e0e0e0;
}

</style>
