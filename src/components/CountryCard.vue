<script setup>
defineProps({
  pais: {
    type: Object,
    required: true
  }
})
</script>

<template>
  <article class="card">
    <img 
      v-bind:src="pais.bandera" 
      v-bind:alt="'Bandera de ' + pais.nombre" 
      class="card-flag"
      loading="lazy"
    />
    <div class="card-body">
      <h2 class="card-title">{{ pais.nombre }}</h2>
      <ul class="card-info">
        <li><strong>Capital:</strong> {{ pais.capital }}</li>
        <li><strong>Región:</strong> {{ pais.region }}</li>
        <li><strong>Población:</strong> {{ pais.poblacion.toLocaleString('es-MX') }} hab.</li>
      </ul>
    </div>
  </article>
</template>