<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Candidate Pipeline</title>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha3/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
  <style>
    body {
      background-color: #f5f7fa;
      font-family: Arial, sans-serif;
    }
    .table {
      background: #ffffff;
      border-radius: 8px;
      box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.1);
    }
    .table thead th {
      background-color: rgba(0, 0, 0, 0.05);
      font-size: 14px;
      text-transform: uppercase;
    }
    .table tbody tr {
      border-bottom: 1px solid #e9ecef;
    }
    .tag {
      padding: 5px 10px;
      border-radius: 12px;
      font-size: 12px;
      color: #ffffff;
      display: inline-block;
    }
    .tag-interviewing {
      background-color: #0d6efd;
    }
    .tag-offer {
      background-color: #198754;
    }
    .tag-rejected {
      background-color: #dc3545;
    }
    .candidate-details {
      position: fixed;
      top: 0;
      right: 0;
      width: 35%;
      height: 100%;
      background-color: #f8f9fa;
      box-shadow: -2px 0 5px rgba(0, 0, 0, 0.1);
      padding: 20px;
      overflow-y: auto;
      transition: transform 0.3s ease;
    }
    .candidate-details.open {
      transform: translateX(0);
    }
    .candidate-details.closed {
      transform: translateX(100%);
    }
    .candidate-avatar {
      width: 100px;
      height: 100px;
      border-radius: 50%;
      margin-bottom: 10px;
    }
    .close-btn {
      float: right;
      font-size: 20px;
      font-weight: bold;
      color: #333;
      text-decoration: none;
    }
    .close-btn:hover {
      color: #000;
    }
  </style>
</head>
<body>
  <div id="app" class="container my-4">
    <h2 class="mb-4">Candidates ({{ candidates.length }})</h2>
    <table class="table">
      <thead>
        <tr>
          <th>Candidates</th>
          <th>LinkedIn</th>
          <th>Candidate Status</th>
          <th>Summary</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="candidate in candidates" :key="candidate.id" @click="showDetails(candidate)">
          <td>{{ candidate.name }}</td>
          <td>
            <a :href="candidate.linkedin" target="_blank" class="text-decoration-none text-primary">LinkedIn</a>
          </td>
          <td>
            <span :class="['tag', getStatusClass(candidate.status)]">
              {{ candidate.status }}
            </span>
          </td>
          <td>{{ candidate.summary }}</td>
        </tr>
      </tbody>
    </table>

    <!-- Панель с деталями -->
    <div :class="['candidate-details', selectedCandidate ? 'open' : 'closed']">
      <a href="#" class="close-btn" @click.prevent="selectedCandidate = null">&times;</a>
      <div v-if="selectedCandidate">
        <img :src="selectedCandidate.avatar" alt="Avatar" class="candidate-avatar img-thumbnail mb-3">
        <h3>{{ selectedCandidate.name }}</h3>
        <p><strong>Status:</strong> {{ selectedCandidate.status }}</p>
        <p><strong>Summary:</strong> {{ selectedCandidate.summary }}</p>
        <p>
          <strong>LinkedIn:</strong>
          <a :href="selectedCandidate.linkedin" target="_blank" class="text-decoration-none text-primary">{{ selectedCandidate.linkedin }}</a>
        </p>
        <p>
          <strong>CV:</strong>
          <a :href="selectedCandidate.cvLink" target="_blank" class="btn btn-outline-primary btn-sm">Download CV</a>
        </p>
      </div>
    </div>
  </div>

  <script>
    new Vue({
      el: "#app",
      data() {
        return {
          candidates: [],
          selectedCandidate: null,
        };
      },
      methods: {
        async fetchCandidates() {
          const API_KEY = "pat94fgVk7Hg9DD2y.8d94f09c0d80ea25aedbf399336d802878e06c5ea7d06935b207e97f480f146a";
          const BASE_ID = "appV7SAgCIQPHKW0f";
          const TABLE_NAME = "tblz4NvNAsiPck0uS";
          const url = `https://api.airtable.com/v0/${BASE_ID}/${TABLE_NAME}`;

          try {
            const response = await axios.get(url, {
              headers: {
                Authorization: `Bearer ${API_KEY}`,
              },
            });

            this.candidates = response.data.records.map((record) => ({
              id: record.id,
              name: record.fields["Full Name"] || "Unknown Name",
              linkedin: record.fields["LinkedIn"] || "#",
              status: record.fields["Candidate Status (Recruiting)"] || "No Status",
              summary: record.fields["Summary"] || "No Summary",
              avatar: record.fields["Avatar"]?.[0]?.url || "https://via.placeholder.com/100",
              cvLink: record.fields["Attach CV and other files"]?.[0] || "#",
            }));
          } catch (error) {
            console.error("Error fetching candidates:", error);
          }
        },
        getStatusClass(status) {
          switch (status.toLowerCase()) {
            case "interviewing":
              return "tag-interviewing";
            case "offer":
              return "tag-offer";
            case "rejected":
              return "tag-rejected";
            default:
              return "tag-default";
          }
        },
        showDetails(candidate) {
          this.selectedCandidate = candidate;
        },
      },
      mounted() {
        this.fetchCandidates();
      },
    });
  </script>
</body>
</html>
