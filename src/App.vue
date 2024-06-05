<template>
  <div class="App">
    <h1>LohntNicht</h1>
    <AddProject :projects="projects" @update-projects="updateProjects" />
    <AddHours :projects="projects" @update-projects="updateProjects" />
    <ProjectList :projects="projects" @update-projects="updateProjects" />
    <div>
      <h2>Gesamtsumme</h2>
      <p>Normale Zeit: {{ totalTime }}</p>
      <p>Industriestunden: {{ totalIndustrialHours }}</p>
      <div>
        <button @click="exportData('csv')">Export as CSV</button>
        <button @click="exportData('json')">Export as JSON</button>
        <button @click="resetApp">Reset App</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import AddProject from './components/AddProject.vue';
import AddHours from './components/AddHours.vue';
import ProjectList from './components/ProjectList.vue';

export default {
  components: {
    AddProject,
    AddHours,
    ProjectList
  },
  setup() {
    const projects = ref([]);
    const totalTime = ref('0:00');
    const totalIndustrialHours = ref('0.00');

    onMounted(() => {
      const storedProjects = JSON.parse(localStorage.getItem('projects'));
      if (storedProjects) {
        projects.value = storedProjects;
        calculateTotals();
      }
    });

    const updateProjects = (updatedProjects) => {
      projects.value = updatedProjects;
      localStorage.setItem('projects', JSON.stringify(updatedProjects));
      calculateTotals();
    };

    const addTimes = (time1, time2) => {
      if (typeof time1 !== 'string') time1 = '0:00';
      if (typeof time2 !== 'string') time2 = '0:00';
      const [hours1, minutes1] = time1.split(':').map(Number);
      const [hours2, minutes2] = time2.split(':').map(Number);
      const totalMinutes = (hours1 * 60 + minutes1) + (hours2 * 60 + minutes2);
      const totalHours = Math.floor(totalMinutes / 60);
      const remainingMinutes = totalMinutes % 60;
      return `${totalHours}:${remainingMinutes < 10 ? '0' : ''}${remainingMinutes}`;
    };

    const subtractTimes = (time1, time2) => {
      if (typeof time1 !== 'string') time1 = '0:00';
      if (typeof time2 !== 'string') time2 = '0:00';
      const [hours1, minutes1] = time1.split(':').map(Number);
      const [hours2, minutes2] = time2.split(':').map(Number);
      const totalMinutes1 = hours1 * 60 + minutes1;
      const totalMinutes2 = hours2 * 60 + minutes2;
      const totalMinutes = totalMinutes1 - totalMinutes2;
      const totalHours = Math.floor(totalMinutes / 60);
      const remainingMinutes = Math.abs(totalMinutes % 60);
      return `${totalHours}:${remainingMinutes < 10 ? '0' : ''}${remainingMinutes}`;
    };

    const calculateTotals = () => {
      let totalAddMinutes = 0;
      let totalSubtractMinutes = 0;

      projects.value.forEach(project => {
        const [hours, minutes] = project.hours.split(':').map(Number);
        const projectMinutes = hours * 60 + minutes;

        if (project.operation === 'add') {
          totalAddMinutes += projectMinutes;
        } else {
          totalSubtractMinutes += projectMinutes;
        }
      });

      const netMinutes = totalAddMinutes - totalSubtractMinutes;
      const totalHours = Math.floor(Math.abs(netMinutes) / 60);
      const remainingMinutes = Math.abs(netMinutes) % 60;
      totalTime.value = `${netMinutes >= 0 ? '' : '-'}${totalHours}:${remainingMinutes < 10 ? '0' : ''}${remainingMinutes}`;

      totalIndustrialHours.value = convertToIndustrialHours(netMinutes);
    };

    const convertToIndustrialHours = (totalMinutes) => {
      const industrialHours = totalMinutes / 60;
      return industrialHours.toFixed(2);
    };

    const exportData = (format) => {
      const data = projects.value.map(project => ({
        name: project.name,
        hours: project.hours,
        industrialHours: convertToIndustrialHours(project.hours.split(':').map(Number).reduce((acc, val) => acc * 60 + val)),
        operation: project.operation
      }));

      const totals = {
        totalTime: totalTime.value,
        totalIndustrialHours: totalIndustrialHours.value
      };

      if (format === 'csv') {
        exportCSV(data, totals);
      } else {
        exportJSON(data, totals);
      }
    };

    const exportCSV = (data, totals) => {
      let csvContent = "data:text/csv;charset=utf-8,";
      csvContent += "Name,Hours,Industrial Hours,Operation\n";
      data.forEach(row => {
        csvContent += `${row.name},${row.hours},${row.industrialHours},${row.operation}\n`;
      });
      csvContent += `Total,${totals.totalTime},${totals.totalIndustrialHours}\n`;

      const encodedUri = encodeURI(csvContent);
      const link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", "projects.csv");
      document.body.appendChild(link);

      link.click();
      document.body.removeChild(link);
    };

    const exportJSON = (data, totals) => {
      const jsonData = {
        projects: data,
        totals: totals
      };

      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(jsonData));
      const link = document.createElement("a");
      link.setAttribute("href", dataStr);
      link.setAttribute("download", "projects.json");
      document.body.appendChild(link);

      link.click();
      document.body.removeChild(link);
    };

    const resetApp = () => {
      projects.value = [];
      totalTime.value = '0:00';
      totalIndustrialHours.value = '0.00';
      localStorage.removeItem('projects');
    };

    return {
      projects,
      totalTime,
      totalIndustrialHours,
      updateProjects,
      addTimes,
      subtractTimes,
      calculateTotals,
      convertToIndustrialHours,
      exportData,
      resetApp
    };
  }
};
</script>

<style>
.App {
  font-family: sans-serif;
  text-align: center;
}

input, select, button {
  margin: 5px;
}

button {
  margin-left: 10px;
}
</style>
