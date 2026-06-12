<template>
  <div class="container">
    <div v-if="loading" class="loading">Cargando datos de la constancia...</div>
    
    <div v-else-if="error" class="error">{{ error }}</div>

    <div v-else-if="enrollmentData.length > 0" class="certificate-box">
      <header class="header">
        <h2>CONSTANCIA DE MATRÍCULA DE LABORATORIO</h2>
        <h3>Escuela Profesional de Ingeniería de Sistemas EPIS</h3>
        <p class="date">Emitido el: {{ formattedDate }}</p>
      </header>
      
      <hr>
      
      <section class="section">
        <h4>DATOS DEL ALUMNO</h4>
        <table class="table-info">
          <tbody>
            <tr>
              <td><strong>C.U.I.:</strong></td>
              <td>{{ enrollmentData[0].student.cui }}</td>
            </tr>
            <tr>
              <td><strong>Nombre completo:</strong></td>
              <td>{{ enrollmentData[0].student.full_name }}</td>
            </tr>
            <tr>
              <td><strong>Email:</strong></td>
              <td>{{ enrollmentData[0].student.email }}</td>
            </tr>
          </tbody>
        </table>
      </section>
      
      <section class="section">
        <h4>ASIGNATURAS MATRICULADAS</h4>
        <table class="table-data">
          <thead>
            <tr>
              <th>N°</th>
              <th>Código</th>
              <th>Curso</th>
              <th>Año</th>
              <th>Grupo</th>
              <th>Laboratorio</th>
              <th>Docente</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in enrollmentData" :key="item.id">
              <td>{{ index + 1 }}</td>
              <td>{{ item.workload.course.code }}</td>
              <td>
                <strong>{{ item.workload.course.name }}</strong> 
                ({{ item.workload.course.acronym }})
              </td>
              <td>{{ item.workload.course.year_display }}</td>
              <td>{{ item.workload.group }}</td>
              <td>{{ item.workload.laboratory }}</td>
              <td>{{ item.workload.teacher.full_name || 'Por asignar' }}</td>
            </tr>
          </tbody>
        </table>
      </section>
      
      <footer class="footer">
        <p><strong>Total de cursos matriculados:</strong> {{ enrollmentData.length }}</p>
        <p class="digital-sign">
          Documento generado digitalmente por el Sistema de Matrícula de Laboratorio EPIS.
        </p>
      </footer>
    </div>

    <div v-else class="no-data">No se encontraron registros para el CUI especificado.</div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import axios from 'axios'

const route = useRoute()
const enrollmentData = ref([])
const loading = ref(true)
const error = ref(null)

const apiBaseUrl = import.meta.env.VITE_API_BASE_URL

// Función para formatear la fecha actual al estilo DD/MM/AAAA
const formattedDate = computed(() => {
  const date = new Date()
  const day = String(date.getDate()).padStart(2, '0')
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const year = date.getFullYear()
  return `${day}/${month}/${year}`
})

const fetchConstancia = async () => {
  try {
    const cui = route.params.cui
    const response = await axios.get(`${apiBaseUrl}/enrollment-certificate/`, {
      params: { cui: cui }
    })
    enrollmentData.value = response.data.results
  } catch (err) {
    console.error(err)
    error.value = 'Hubo un error al conectar con el servidor o cargar los datos.'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchConstancia()
})
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background-color: #f0f2f5;
  padding: 20px;
  font-family: 'Helvetica Neue', Arial, sans-serif;
  box-sizing: border-box;
}

.certificate-box {
  background-color: #ffffff;
  width: 100%;
  max-width: 900px;
  padding: 40px;
  border-radius: 4px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  border: 1px solid #dcdfe6;
}

.header {
  text-align: center;
  margin-bottom: 20px;
}

.header h2 {
  font-size: 1.4rem;
  color: #1a1a1a;
  margin: 0 0 5px 0;
  letter-spacing: 0.5px;
}

.header h3 {
  font-size: 1.1rem;
  color: #4a4a4a;
  font-weight: normal;
  margin: 0 0 15px 0;
}

.date {
  font-size: 0.9rem;
  color: #777;
  margin: 0;
}

hr {
  border: 0;
  border-top: 2px solid #333;
  margin-bottom: 30px;
}

.section {
  margin-bottom: 30px;
}

.section h4 {
  font-size: 1rem;
  color: #0056b3;
  margin: 0 0 12px 0;
  border-bottom: 1px solid #0056b3;
  padding-bottom: 4px;
  letter-spacing: 0.5px;
}

.table-info {
  width: 100%;
  border-collapse: collapse;
}

.table-info td {
  padding: 6px 0;
  font-size: 0.95rem;
}

.table-info td:first-child {
  width: 150px;
  color: #555;
}

.table-data {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  font-size: 0.9rem;
}

.table-data th, .table-data td {
  border: 1px solid #dcdfe6;
  padding: 10px;
  text-align: left;
}

.table-data th {
  background-color: #f5f7fa;
  color: #333;
  font-weight: bold;
}

.table-data tr:nth-child(even) {
  background-color: #fafafa;
}

.footer {
  margin-top: 40px;
  font-size: 0.95rem;
}

.footer p {
  margin: 5px 0;
}

.digital-sign {
  margin-top: 25px !important;
  font-size: 0.8rem;
  color: #909399;
  font-style: italic;
  text-align: center;
  border-top: 1px dashed #dcdfe6;
  padding-top: 15px;
}

.loading, .no-data {
  font-size: 1.1rem;
  color: #606266;
  text-align: center;
}

.error {
  color: #f56c6c;
  background-color: #fef0f0;
  border: 1px solid #fde2e2;
  padding: 15px;
  border-radius: 4px;
  font-weight: bold;
  text-align: center;
}
</style>
