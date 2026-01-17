<script setup>
defineProps({
  documents: {
    type: Array,
    required: true
  }
})

const truncateDescription = (desc) => {
  if (!desc) return 'Sin descripción'
  const maxLength = 60
  return desc.length > maxLength ? desc.substring(0, maxLength) + '...' : desc
}

const truncateTitle = (title) => {
  if (!title) return 'Sin título'
  const maxLength = 50
  return title.length > maxLength ? title.substring(0, maxLength) + '...' : title
}

const truncateAuthor = (author) => {
  if (!author) return 'Desconocido'
  const maxLength = 35
  return author.length > maxLength ? author.substring(0, maxLength) + '...' : author
}
</script>

<template>
  <div class="table-container">
    <table v-if="documents.length > 0">
      <thead>
        <tr>
          <th class="col-id">#</th>
          <th class="col-titulo">Título</th>
          <th class="col-autor">Autor</th>
          <th class="col-fecha">Fecha</th>
          <th class="col-descripcion">Resumen</th>
          <th class="col-plos">PLOS</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="doc in documents" :key="doc.id">
          <td class="col-id">{{ doc.id }}</td>
          <td class="col-titulo" :title="doc.titulo"><strong>{{ truncateTitle(doc.titulo) }}</strong></td>
          <td class="col-autor" :title="doc.autor"><small>{{ truncateAuthor(doc.autor) }}</small></td>
          <td class="col-fecha">{{ doc.fecha }}</td>
          <td class="col-descripcion" :title="doc.descripcion"><small>{{ truncateDescription(doc.descripcion) }}</small></td>
          <td class="col-plos">
            <a v-if="doc.plos" :href="`https://journals.plos.org/plosone/article?id=${doc.plos}`" target="_blank" class="plos-link" title="Ver en PLOS">
              🔗
            </a>
          </td>
        </tr>
      </tbody>
    </table>
    
    <div v-else class="no-data">
      No se encontraron documentos
    </div>
  </div>
</template>

<style scoped>
.table-container {
  overflow-x: auto;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

table {
  width: 100%;
  border-collapse: collapse;
  background-color: #1a1a1a;
  font-size: 0.85rem;
}

thead {
  background-color: #646cff;
  position: sticky;
  top: 0;
}

th {
  padding: 0.6rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  white-space: nowrap;
}

td {
  padding: 0.5rem 0.6rem;
  border-bottom: 1px solid #333;
}

tbody tr {
  transition: background-color 0.2s;
}

tbody tr:hover {
  background-color: #2a2a2a;
}

tbody tr:last-child td {
  border-bottom: none;
}

.col-id {
  width: 45px;
  min-width: 45px;
  text-align: center;
  font-weight: 600;
}

.col-titulo {
  width: 30%;
  min-width: 180px;
}

.col-autor {
  width: 20%;
  min-width: 120px;
}

.col-fecha {
  width: 90px;
  min-width: 90px;
}

.col-descripcion {
  width: 35%;
  min-width: 180px;
  color: #aaa;
  line-height: 1.2;
}

.col-plos {
  width: 50px;
  min-width: 50px;
  text-align: center;
}

.plos-link {
  color: #646cff;
  text-decoration: none;
  font-size: 1rem;
  transition: all 0.2s;
  display: inline-block;
}

.plos-link:hover {
  color: #535bf2;
  transform: scale(1.3);
}

.no-data {
  text-align: center;
  padding: 3rem;
  font-size: 1.2rem;
  color: #888;
}

small {
  font-size: 0.8rem;
}

@media (prefers-color-scheme: light) {
  table {
    background-color: #ffffff;
  }
  
  td {
    border-bottom-color: #e0e0e0;
  }
  
  tbody tr:hover {
    background-color: #f5f5f5;
  }
  
  .col-descripcion {
    color: #666;
  }
}
</style>
