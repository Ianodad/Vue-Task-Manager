<template>
  <div v-show="showAddTask">
    <AddTask @add-task="addTask" />
  </div>
  <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
</template>

<script>
import Tasks from '../components/Tasks.vue';
import AddTask from '../components/AddTask.vue';

export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: {
    Tasks,
    AddTask,
  },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      const res = await fetch('/api/tasks', {
        method: 'POST',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(task),
      });

      const data = await res.json();
      this.tasks = [...this.tasks, data];
    },
    async deleteTask(id) {
      // eslint-disable-next-line no-restricted-globals
      const res = await fetch(`api/tasks/${id}`, {
        method: 'DELETE',
      });

      // eslint-disable-next-line no-unused-expressions
      res.status === 200 ? (this.tasks = this.tasks.filter((task) => task.id !== id)) : console.log('Error deleting task');
      // this.tasks = this.tasks.filter((task) => task.id !== id);
      console.log('Task', id);
    },

    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder };

      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(updTask),
      });

      const data = await res.json();

      this.tasks = this.tasks.map((task) =>
        // eslint-disable-next-line implicit-arrow-linebreak
        (task.id === id
          ? {
            ...task,
            reminder: data.reminder,
          }
          : task));
      console.log('Reminder', id);
    },
    async fetchTasks() {
      const res = await fetch('api/tasks');

      const data = await res.json();

      return data;
    },
    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);

      const data = await res.json();

      return data;
    },

  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
