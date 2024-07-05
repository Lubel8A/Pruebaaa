<template>
  <div class="container">
    <input type="file" @change="handleFileUpload" accept=".xlsx" class="file-input">
    <div v-if="loading" class="loading-message">Cargando...</div>
    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
    <div class="table-container">
      <table v-if="jsonData.length" class="data-table">
        <thead>
          <tr>
            <th v-for="(col, index) in columns" :key="index">{{ col }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in paginatedData" :key="rowIndex">
            <td v-for="(col, colIndex) in columns" :key="colIndex">{{ row[col] }}</td>
          </tr>
        </tbody>
      </table>
      <div v-else class="no-data-message">No hay datos para mostrar.</div>
    </div>
    <div class="pagination">
      <div class="pagination-info">
        Mostrando {{ (currentPage - 1) * rowsPerPage + 1 }} - {{ Math.min(currentPage * rowsPerPage, jsonData.length) }} de {{ jsonData.length }} registros
      </div>
      <button @click="prevPage" :disabled="currentPage === 1" class="pagination-button">Anterior</button>
      <button @click="nextPage" :disabled="currentPage === totalPages" class="pagination-button">Siguiente</button>
    </div>
    <button @click="sendData" :disabled="!jsonData.length || loading" class="send-button">Enviar Datos</button>
  </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      file: null,
      jsonData: [],
      columns: [],
      loading: false,
      errorMessage: '',
      currentPage: 1,
      rowsPerPage: 20,
    };
  },
  computed: {
    paginatedData() {
      const start = (this.currentPage - 1) * this.rowsPerPage;
      const end = start + this.rowsPerPage;
      return this.jsonData.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.jsonData.length / this.rowsPerPage);
    },
  },
  methods: {
    handleFileUpload(event) {
      const file = event.target.files[0];
      if (!file || !file.name.endsWith('.xlsx')) {
        this.errorMessage = 'Por favor, sube un archivo Excel (.xlsx).';
        return;
      }
      this.errorMessage = '';
      this.loading = true;

      const reader = new FileReader();
      reader.onload = (e) => {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: 'array' });
        const firstSheetName = workbook.SheetNames[0];
        const worksheet = workbook.Sheets[firstSheetName];
        const jsonData = XLSX.utils.sheet_to_json(worksheet, { defval: '' });

        this.columns = Object.keys(jsonData[0]);
        this.jsonData = jsonData;
        this.loading = false;
      };
      reader.readAsArrayBuffer(file);
    },
    async sendData() {
      this.loading = true;
      try {
        const response = await fetch('https://pruebaSubida/Excel', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(this.jsonData),
        });
        if (!response.ok) {
          throw new Error('Error al enviar los datos.');
        }
        alert('Datos enviados correctamente.');
      } catch (error) {
        this.errorMessage = 'Hubo un error al enviar los datos.';
        console.error('Error al enviar datos:', error);
      } finally {
        this.loading = false;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
  background-color: #f0f0f0;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.file-input {
  margin-bottom: 16px;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: #fff;
  cursor: pointer;
}

.loading-message {
  margin-bottom: 16px;
  font-size: 1.2rem;
}

.error-message {
  margin-bottom: 16px;
  color: #ff6347;
  font-size: 1.2rem;
}

.table-container {
  max-height: 400px;
  overflow-y: auto;
  overflow-x: auto;
  margin-bottom: 16px;
  background-color: #fff; /* Fondo para la tabla */
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
}

.data-table th, .data-table td {
  border: 1px solid #ddd;
  padding: 12px;
}

.no-data-message {
  margin: 16px;
  font-size: 1.2rem;
  color: #555;
}

.pagination {
  margin-top: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pagination-info {
  margin-right: 20px;
}

.pagination-button {
  padding: 10px 20px;
  margin-right: 8px;
  border: none;
  background-color: #007bff;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.pagination-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.send-button {
  margin-top: 16px;
  padding: 10px 20px;
  border: none;
  background-color: #28a745;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.send-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.send-button:hover, .pagination-button:hover {
  background-color: #0056b3;
}
</style>
