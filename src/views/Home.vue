<template>
  <div v-show="showAddTask">
    <AddTask @add-task="addTask" />
  </div>
  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>

<script>
import Tasks from '../components/Tasks.vue';
import AddTask from '../components/Add-task.vue';

export default {
  name: 'Home',
  props: {
    showAddTask: Boolean,
  },
  components: { Tasks, AddTask },
  data() {
    return {
      tasks: [],
    };
  },
  methods: {
    async addTask(task) {
      console.log(task);
      const res = await fetch('api/tasks', {
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
      console.log(id, 'appvue');
      if (confirm('are sure you want to delete this task?')) {
        const res = await fetch(`api/tasks/${id}`, {
          method: 'DELETE',
        });
        if (res.status === 200) {
          this.tasks = this.tasks.filter(task => task.id !== id);
        } else {
          alert('Error deleting task');
        }
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTaskById(id);

      const updatedTasks = {
        ...taskToToggle,
        reminder: !taskToToggle.reminder,
      };

      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json',
        },
        body: JSON.stringify(updatedTasks),
      });
      const data = await res.json();

      this.tasks.forEach(task => {
        if (id === task.id) {
          task.reminder = data.reminder;
        }
      });
    },
    async fetchTasks() {
      const res = await fetch(`api/tasks`);
      const data = res.json();
      return data;
    },
    async fetchTaskById(id) {
      const res = await fetch(`api/tasks/${id}`);
      const data = res.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>
