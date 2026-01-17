<script setup>
import { ref, computed, onMounted } from 'vue'
import jsPDF from 'jspdf'
import 'jspdf-autotable'
import DocumentTable from './components/DocumentTable.vue'

const documents = ref([])
const searchQuery = ref('')
const loading = ref(false)
const error = ref(null)
const currentPage = ref(1)
const itemsPerPage = 15

// Cargar documentos desde la API de PLOS
const loadDocumentsFromAPI = async () => {
  loading.value = true
  error.value = null
  try {
    const response = await fetch('https://api.plos.org/search?q=title&rows=100')
    const data = await response.json()
    
    // Mapear los datos de la API al formato de nuestra tabla
    if (data.response && data.response.docs) {
      documents.value = data.response.docs.map((doc, index) => ({
        id: index + 1,
        titulo: doc.title_display || 'Sin título',
        autor: doc.author_display ? doc.author_display.join(', ') : 'Autor desconocido',
        fecha: doc.publication_date ? new Date(doc.publication_date).toLocaleDateString('es-ES') : 'Fecha desconocida',
        descripcion: Array.isArray(doc.abstract) && doc.abstract.length > 0 ? doc.abstract[0].trim() : 'Sin descripción disponible',
        plos: doc.id || 'N/A',
        url: doc.url || ''
      }))
    }
  } catch (err) {
    error.value = 'Error al cargar los documentos: ' + err.message
    console.error('Error:', err)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  // Cargar los documentos desde la API
  loadDocumentsFromAPI()
})

// Filtrar documentos basado en la búsqueda
const filteredDocuments = computed(() => {
  if (!searchQuery.value) {
    return documents.value
  }
  
  const query = searchQuery.value.toLowerCase()
  return documents.value.filter(doc => {
    return (
      doc.titulo?.toLowerCase().includes(query) ||
      doc.autor?.toLowerCase().includes(query) ||
      doc.descripcion?.toLowerCase().includes(query)
    )
  })
})

// Obtener documentos paginados
const paginatedDocuments = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredDocuments.value.slice(start, end)
})

// Calcular total de páginas
const totalPages = computed(() => {
  return Math.ceil(filteredDocuments.value.length / itemsPerPage)
})

// Reiniciar a página 1 cuando se busca
const handleSearch = () => {
  currentPage.value = 1
}

// Función para descargar PDF
const downloadPDF = () => {
  const doc = new jsPDF()
  
  // Título del documento
  doc.setFontSize(18)
  doc.text('Listado de Documentos - PLOS', 14, 22)
  
  // Información adicional
  doc.setFontSize(11)
  doc.text(`Total de documentos: ${filteredDocuments.value.length}`, 14, 30)
  doc.text(`Generado el: ${new Date().toLocaleDateString('es-ES')}`, 14, 36)
  
  // Preparar datos para la tabla
  const tableData = filteredDocuments.value.map(doc => [
    doc.id,
    doc.titulo,
    doc.autor,
    doc.fecha,
    doc.descripcion.substring(0, 100) + '...'
  ])
  
  // Crear tabla
  doc.autoTable({
    startY: 42,
    head: [['ID', 'Título', 'Autor', 'Fecha', 'Descripción']],
    body: tableData,
    theme: 'grid',
    styles: {
      fontSize: 8,
      cellPadding: 3
    },
    headStyles: {
      fillColor: [100, 108, 255],
      textColor: 255,
      fontStyle: 'bold'
    },
    columnStyles: {
      0: { cellWidth: 15 },
      1: { cellWidth: 40 },
      2: { cellWidth: 30 },
      3: { cellWidth: 25 },
      4: { cellWidth: 50 }
    }
  })
  
  // Guardar el PDF
  doc.save(`documentos_plos_${new Date().getTime()}.pdf`)
}

// Función para cargar archivo JSON personalizado
const loadCustomJSON = (event) => {
  const file = event.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (e) => {
      try {
        const data = JSON.parse(e.target.result)
        documents.value = Array.isArray(data) ? data : [data]
      } catch (error) {
        alert('Error al leer el archivo JSON: ' + error.message)
      }
    }
    reader.readAsText(file)
  }
}
</script>

<template>
  <div>
    <h1>📚 Gestor de Documentos - PLOS API</h1>
    
    <div v-if="error" class="error-message">
      ⚠️ {{ error }}
      <button @click="loadDocumentsFromAPI" class="btn" style="margin-left: 1rem;">
        🔄 Reintentar
      </button>
    </div>
    
    <div v-if="loading" class="loading">
      ⏳ Cargando documentos desde PLOS...
    </div>
    
    <template v-if="!loading && documents.length > 0">
      <div class="controls">
        <input
          v-model="searchQuery"
          @input="handleSearch"
          type="text"
          class="search-box"
          placeholder="🔍 Buscar por título, autor o descripción..."
        />
        
        <button @click="downloadPDF" class="btn">
          📥 Descargar PDF
        </button>
        
        <button @click="loadDocumentsFromAPI" class="btn btn-secondary">
          🔄 Recargar
        </button>
      </div>
      
      <DocumentTable :documents="paginatedDocuments" />
      
      <!-- Paginación -->
      <div class="pagination">
        <button 
          @click="currentPage--" 
          :disabled="currentPage === 1"
          class="btn btn-pagination"
        >
          ← Anterior
        </button>
        
        <div class="pagination-info">
          Página {{ currentPage }} de {{ totalPages }}
        </div>
        
        <button 
          @click="currentPage++" 
          :disabled="currentPage === totalPages"
          class="btn btn-pagination"
        >
          Siguiente →
        </button>
      </div>
      
      <div class="stats">
        Mostrando {{ paginatedDocuments.length }} de {{ filteredDocuments.length }} documentos (Total: {{ documents.length }})
      </div>
    </template>
  </div>
</template>
