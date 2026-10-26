<script setup>
const busqueda = defineModel('busqueda')
const region = defineModel('region')

const regiones = ['Africa', 'Americas', 'Asia', 'Europe', 'Oceania']
</script>

<template>
  <section class="filtros-container">
    <div class="input-group">
      <label for="busqueda">🔎 Buscar país:</label>
      <input 
        id="busqueda"
        type="text" 
        v-model="busqueda" 
        placeholder="Ej. Mexico, Canada, Japan..."
        class="input-busqueda"
      />
    </div>

    <div class="input-group">
      <label for="region">🌍 Filtrar por región:</label>
      <select id="region" v-model="region" class="select-region">
        <option value="">Todas las regiones</option>
        <option v-for="reg in regiones" :key="reg" :value="reg">
          {{ reg }}
        </option>
      </select>
    </div>
  </section>
</template>