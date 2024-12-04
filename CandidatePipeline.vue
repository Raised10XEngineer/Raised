<template>
  <div class="candidate-pipeline">
    <h2>{{ currentStage }} ({{ filteredCandidates.length }})</h2>
    <!-- Таблица -->
    <table>
      <thead>
        <tr>
          <th>Candidates</th>
          <th>LinkedIn</th>
          <th>CV</th>
          <th>Candidate Status</th>
          <th>Summary</th>
          <th>Scoring based on criteria</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="candidate in filteredCandidates"
          :key="candidate.id"
          @click="showDetails(candidate)"
          class="candidate-row"
        >
          <td>{{ candidate.name }}</td>
          <td><a :href="candidate.linkedin" target="_blank">LinkedIn</a></td>
          <td><a :href="candidate.cvLink" target="_blank">CV</a></td>
          <td>{{ candidate.status }}</td>
          <td>{{ candidate.summary }}</td>
          <td>{{ candidate.scoring }}</td>
        </tr>
      </tbody>
    </table>
    <!-- Панель деталей -->
    <div v-bind:class="['candidate-details', { open: selectedCandidate }]">
      <div v-if="selectedCandidate">
        <h3>{{ selectedCandidate.name }}</h3>
        <div class="overview">
          <h4>Overview</h4>
          <p><strong>Summary:</strong> {{ selectedCandidate.summary }}</p>
          <p>
            <strong>LinkedIn:</strong>
            <a :href="selectedCandidate.linkedin" target="_blank">{{ selectedCandidate.linkedin }}</a>
          </p>
          <p>
            <strong>Avatar:</strong>
          </p>
          <img :src="selectedCandidate.avatar" alt="Avatar" class="avatar" />
          <p>
            <strong>Attach CV and other files:</strong>
          </p>
          <a :href="selectedCandidate.cvLink" target="_blank">{{ selectedCandidate.cvName }}</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

const API_KEY = "pat94fgVk7Hg9DD2y.8d94f09c0d80ea25aedbf399336d802878e06c5ea7d06935b207e97f480f146a";
const BASE_ID = "appV7SAgCIQPHKW0f";
const TABLE_NAME = "tblz4NvNAsiPck0uS";

export default {
  data() {
    return {
      currentStage: "Interviewing",
      stages: ["Screening", "Interviewing", "Offer", "Closed lost", "All candidates"],
      candidates: [], // Пустой массив для данных из Airtable
      selectedCandidate: null,
    };
  },
  computed: {
    filteredCandidates() {
      if (this.currentStage === "All candidates") {
        return this.candidates;
      }
      return this.candidates.filter((candidate) => candidate.status === this.currentStage);
    },
  },
  methods: {
    async fetchCandidates() {
      try {
        const response = await axios.get(
          `https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}`,
          {
            headers: {
              Authorization: `Bearer ${API_KEY}`,
            },
          }
        );
        // Парсим данные из Airtable
        this.candidates = response.data.records.map((record) => ({
  id: record.id,
  name: Array.isArray(record.fields["Full Name"]) ? record.fields["Full Name"][0] : record.fields["Full Name"], // Извлекаем первую запись из массива
  linkedin: Array.isArray(record.fields["LinkedIn"]) ? record.fields["LinkedIn"][0] : record.fields["LinkedIn"], // Проверяем, если это массив
  cvLink: Array.isArray(record.fields["Attach CV and other files"]) ? record.fields["Attach CV and other files"][0] : record.fields["Attach CV and other files"],
  cvName: "Download CV",
  status: Array.isArray(record.fields["Candidate Status (Recruiting)"]) ? record.fields["Candidate Status (Recruiting)"][0] : record.fields["Candidate Status (Recruiting)"],
  summary: Array.isArray(record.fields["Summary"]) ? record.fields["Summary"][0] : record.fields["Summary"],
  scoring: Array.isArray(record.fields["Scoring based on criteria"]) ? record.fields["Scoring based on criteria"][0] : record.fields["Scoring based on criteria"],
  avatar: record.fields["Avatar"]?.[0]?.url || "https://via.placeholder.com/150", // Извлекаем первую запись, если это массив
}));

      } catch (error) {
        console.error("Ошибка загрузки данных из Airtable:", error);
      }
    },
    showDetails(candidate) {
      this.selectedCandidate = candidate;
    },
  },
  mounted() {
    this.fetchCandidates(); // Загрузка данных при монтировании компонента
  },
};
</script>

<style scoped>
.candidate-pipeline {
  margin: 20px;
}
table {
  width: 100%;
  border-collapse: collapse;
  cursor: pointer;
}
th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
th {
  background-color: #F2F2F2;
}
.candidate-row:hover {
  background-color: #F9F9F9;
}
.candidate-details {
  position: fixed;
  right: 0;
  top: 0;
  width: 30%;
  height: 100%;
  background-color: #fff;
  box-shadow: -2px 0 5px rgba(0, 0, 0, 0.1);
  overflow-y: auto;
  transition: transform 0.3s ease;
  transform: translateX(100%);
}
.candidate-details.open {
  transform: translateX(0);
}
.avatar {
  width: 150px;
  height: 150px;
  border-radius: 50%;
}
</style>
