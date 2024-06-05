<template>
  <div>
    <h2>Projekte</h2>
    <ul>
      <li v-for="(project, index) in projects" :key="index">
        {{ project.name }} - {{ project.hours }}
        <select v-model="project.operation" @change="updateOperation(index, project.operation)">
          <option value="add">Addieren</option>
          <option value="subtract">Subtrahieren</option>
        </select>
        <button @click="deleteProject(project.name)">Projekt löschen</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ['projects'],
  methods: {
    deleteProject(projectName) {
      const updatedProjects = this.projects.filter(project => project.name !== projectName);
      this.$emit('update-projects', updatedProjects);
    },
    updateOperation(index, operation) {
      const updatedProjects = [...this.projects];
      updatedProjects[index].operation = operation;
      this.$emit('update-projects', updatedProjects);
    }
  }
};
</script>
