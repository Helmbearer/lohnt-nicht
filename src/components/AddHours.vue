<template>
  <div>
    <h2>Stunden hinzufügen</h2>
    <select v-model="hoursEntry.project">
      <option value="" disabled>Projekt auswählen</option>
      <option v-for="project in projects" :key="project.name" :value="project.name">
        {{ project.name }}
      </option>
    </select>
    <input
        type="text"
        v-model="hoursEntry.hours"
        placeholder="hh:mm oder h,m"
    />
    <button @click="addHours">Stunden hinzufügen</button>
  </div>
</template>

<script>
export default {
  props: ['projects'],
  data() {
    return {
      hoursEntry: { project: '', hours: '0:00' }
    };
  },
  methods: {
    addHours() {
      const projectIndex = this.projects.findIndex(p => p.name === this.hoursEntry.project);
      if (projectIndex !== -1) {
        const updatedProjects = [...this.projects];
        updatedProjects[projectIndex].hours = this.addTimes(updatedProjects[projectIndex].hours, this.formatTime(this.hoursEntry.hours));
        this.$emit('update-projects', updatedProjects);
      }
      this.hoursEntry = { project: '', hours: '0:00' };
    },
    formatTime(input) {
      // Falls die Eingabe bereits im hh:mm Format ist
      if (input.includes(':')) {
        return input;
      }
      // Falls die Eingabe im h,m Format ist
      if (input.includes(',')) {
        let [hours, minutes] = input.split(',').map(Number);
        if (minutes < 10) {
          minutes *= 10;
        }
        return `${hours}:${minutes < 10 ? '0' : ''}${minutes}`;
      }
      // Falls die Eingabe nur eine Zahl ist
      const number = Number(input);
      if (!isNaN(number)) {
        return `${number}:00`;
      }
      return '0:00';
    },
    addTimes(time1, time2) {
      if (typeof time1 !== 'string') time1 = '0:00';
      if (typeof time2 !== 'string') time2 = '0:00';
      const [hours1, minutes1] = time1.split(':').map(Number);
      const [hours2, minutes2] = time2.split(':').map(Number);
      const totalMinutes = (hours1 * 60 + minutes1) + (hours2 * 60 + minutes2);
      const totalHours = Math.floor(totalMinutes / 60);
      const remainingMinutes = totalMinutes % 60;
      return `${totalHours}:${remainingMinutes < 10 ? '0' : ''}${remainingMinutes}`;
    }
  }
};
</script>
