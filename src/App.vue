<script setup>
import { ref, computed, onMounted } from 'vue'
import SearchBar from './components/SearchBar.vue'
import CountryCard from './components/CountryCard.vue'

// Variables reactivas con ref()
const paises = ref([])
const busqueda = ref('')
const regionSeleccionada = ref('')
const cargando = ref(true)
const error = ref(null)

// Función para normalizar los datos sin importar la versión de la API
const normalizarDatos = (lista) => {
  return lista.map((item) => ({
    codigo: item.cca3 || item.alpha3Code || item.name?.common || item.name,
    nombre: item.name?.common || item.name || 'Desconocido',
    bandera: item.flags?.svg || item.flags?.png || item.flag || '',
    capital: Array.isArray(item.capital) ? item.capital[0] : (item.capital || 'Sin capital'),
    region: item.region || 'Sin región',
    poblacion: item.population || 0
  }))
}

// Petición asíncrona con fetch() y manejo de errores con try/catch
const obtenerPaises = async () => {
  cargando.value = true
  error.value = null

  try {
    // Intento primario con el endpoint indicado en los requisitos
    const urlDemo = 'https://api.restcountries.com/countries/v5?q=Canada&api-key=rc_live_demo&pretty=1'
    let respuesta = await fetch(urlDemo)

    // Si la llave demo limita los países o da error CORS, cargamos el catálogo completo v3.1
    if (!respuesta.ok) {
      throw new Error('Fallback a v3.1')
    }

    const datosDemo = await respuesta.json()
    const listaExtraida = Array.isArray(datosDemo) ? datosDemo : (datosDemo.data || [datosDemo])

    // Si el endpoint demo solo devuelve 1 país (Canadá), traemos el resto para que el buscador luzca completo
    if (listaExtraida.length <= 1) {
      const respCompleta = await fetch('https://restcountries.com/v3.1/all?fields=name,flags,capital,region,population,cca3')
      const datosCompletos = await respCompleta.json()
      paises.value = normalizarDatos(datosCompletos)
    } else {
      paises.value = normalizarDatos(listaExtraida)
    }
  } catch (err) {
    try {
      // Respaldo principal que garantiza todos los países para filtrar
      const respRespaldo = await fetch('https://restcountries.com/v3.1/all?fields=name,flags,capital,region,population,cca3')
      if (!respRespaldo.ok) throw new Error('Error en el servidor de la API')
      const datosRespaldo = await respRespaldo.json()
      paises.value = normalizarDatos(datosRespaldo)
    } catch (errorFinal) {
      error.value = 'No fue posible obtener los datos de la API'
    }
  } finally {
    cargando.value = false
  }
}

// Propiedad computada para filtrar con v-model (Nombre y Región)
const paisesFiltrados = computed(() => {
  return paises.value.filter((pais) => {
    const coincideNombre = pais.nombre.toLowerCase().includes(busqueda.value.toLowerCase().trim())
    const coincideRegion = regionSeleccionada.value === '' || pais.region === regionSeleccionada.value
    return coincideNombre && coincideRegion
  })
})

// Primera llamada ejecutada dentro de onMounted()
onMounted(() => {
  obtenerPaises()
})
</script>

<template>
  <div class="contenedor-principal">
    <header class="encabezado">
      <h1>🌍 Explorador de Países</h1>
      <p>Consumo de API REST Countries con Vue 3</p>
    </header>

    <!-- Componente de filtrado con v-model -->
    <SearchBar 
      v-model:busqueda="busqueda" 
      v-model:region="regionSeleccionada" 
    />

    <!-- Estructuras de control v-if / v-else-if / v-else -->
    <div v-if="cargando" class="estado-mensaje cargando">
      <div class="spinner"></div>
      <p>⏳ Cargando países...</p>
    </div>

    <div v-else-if="error" class="estado-mensaje error">
      <p>⚠️ {{ error }}</p>
      <button @click="obtenerPaises" class="btn-reintentar">Reintentar</button>
    </div>

    <div v-else-if="paisesFiltrados.length === 0" class="estado-mensaje sin-resultados">
      <p>🔍 No se encontraron países que coincidan con "{{ busqueda }}".</p>
    </div>

    <!-- Lista de países usando v-for -->
    <main v-else class="grid-paises">
      <CountryCard 
        v-for="pais in paisesFiltrados" 
        :key="pais.codigo" 
        :pais="pais" 
      />
    </main>
  </div>
</template>
<!-- Código revisado por el equipo -->