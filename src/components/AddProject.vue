<template>
  <div>
    <h2>Projekt hinzufügen</h2>
    <input
        type="text"
        v-model="newProjectName"
        placeholder="Projektname"
    />
    <button @click="addProject">Projekt hinzufügen</button>
  </div>
</template>

<script>
export default {
  props: ['projects'],
  data() {
    return {
      newProjectName: ''
    };
  },
  methods: {
    addProject() {
      if (this.newProjectName.trim() === '') return;
      if (this.projects.find(p => p.name === this.newProjectName)) return;
      const updatedProjects = [...this.projects, { name: this.newProjectName, hours: '0:00', operation: 'add' }];
      this.$emit('update-projects', updatedProjects);
      this.newProjectName = '';
    }
  }
};
</script>
