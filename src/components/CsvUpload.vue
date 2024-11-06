<template>
  <div class="container">
    <h2>Upload Your CSV Data</h2>
    <input type="file" @change="handleFileUpload" />

    <!-- Display CSV data as a table -->
    <div v-if="csvData.length > 0">
      <h3>CSV Data</h3>
      <table>
        <thead>
          <tr>
            <th v-for="(value, index) in csvData[0]" :key="index">{{ value }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in csvData.slice(1)" :key="index">
            <td v-for="(value, i) in row" :key="i">{{ value }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Display insights -->
    <div v-if="insights">
      <h3>Dashboard Insights</h3>
      <p><strong>Total Rows:</strong> {{ insights.totalRows }}</p>
      <p><strong>Column Names:</strong> {{ insights.columnNames.join(', ') }}</p>
      <div v-if="insights.numericColumns.length > 0">
        <h4>Numeric Column Statistics</h4>
        <ul>
          <li v-for="(stat, index) in insights.numericColumns" :key="index">
            <strong>{{ stat.column }}:</strong>
            Average = {{ stat.average }}, Sum = {{ stat.sum }},
            Max = {{ stat.max.value }} (Row {{ stat.max.row }}),
            Min = {{ stat.min.value }} (Row {{ stat.min.row }})
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import Papa from 'papaparse';

export default {
  name: 'CsvUpload',
  data() {
    return {
      csvData: [], // Will hold the raw CSV data
      insights: null, // Will hold insights
    };
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (file) {
        Papa.parse(file, {
          complete: (result) => {
            this.csvData = result.data;
            this.analyzeData();
          },
          header: true, // Assuming the first row contains headers
        });
      }
    },
    analyzeData() {
      // Basic insights for now: total rows and column names
      const totalRows = this.csvData.length;
      const columnNames = Object.keys(this.csvData[0]);

      // Analyze numeric columns
      const numericColumns = columnNames
        .filter((col) => !isNaN(this.csvData[1][col]))
        .map((col) => {
          const values = this.csvData
            .map((row, index) => ({ value: parseFloat(row[col]), row: index + 1 }))
            .filter((item) => !isNaN(item.value));

          const sum = values.reduce((acc, item) => acc + item.value, 0);
          const average = sum / values.length;
          const max = values.reduce((prev, curr) => (curr.value > prev.value ? curr : prev));
          const min = values.reduce((prev, curr) => (curr.value < prev.value ? curr : prev));

          return { column: col, average, sum, max, min };
        });

      // Store insights
      this.insights = {
        totalRows,
        columnNames,
        numericColumns,
      };
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h2 {
  font-size: 2em;
  margin-bottom: 20px;
  color: #333;
  text-align: center;
}

input[type="file"] {
  display: block;
  margin: 15px auto;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1em;
}

table {
  width: 100%;
  margin-top: 20px;
  border-collapse: collapse;
  font-size: 0.9em;
  background-color: #f9f9f9;
}

th, td {
  padding: 12px 15px;
  text-align: center;
  border: 1px solid #ddd;
}

th {
  background-color: #007bff;
  color: #fff;
  font-weight: bold;
}

td {
  background-color: #f2f2f2;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}

tr:hover {
  background-color: #f1f1f1;
}

h3, h4 {
  color: #555;
  margin-top: 20px;
  font-weight: bold;
}

ul {
  list-style-type: none;
  padding: 0;
}

ul li {
  margin-bottom: 8px;
  font-size: 1em;
}

strong {
  color: #333;
}
</style>
