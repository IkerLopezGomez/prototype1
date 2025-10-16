<script setup>
import { ref, computed } from 'vue';

const estatDelJoc = ref({
  paraules: [
    { id: 1, text: 'component', estat: 'pendent', errors: 0 },
    { id: 2, text: 'reactivitat', estat: 'pendent', errors: 0 },
    { id: 3, text: 'javascript', estat: 'pendent', errors: 0 },
    { id: 4, text: 'framework', estat: 'pendent', errors: 0 },
    { id: 5, text: 'template', estat: 'pendent', errors: 0 }
  ],
  indexParaulaActiva: 0,
  textEntrat: '',
  estadistiques: [],
  errorglobal: 0,
});

const paraulaActiva = computed(() => {
  return estatDelJoc.value.paraules[estatDelJoc.value.indexParaulaActiva];
});

let tempsIniciParaula = 0;

function iniciarCronometreParaula() {
  tempsIniciParaula = Date.now();
}

// 🔹 Solo devuelve la clase, no modifica errores
function obtenirClasseLletra(lletra, index) {
  const entrada = estatDelJoc.value.textEntrat[index];
  if (!entrada) return '';
  return lletra === entrada ? 'lletra-correcta' : 'lletra-incorrecta';
}

// 🔹 Función principal
function validarProgres() {
  const entrada = estatDelJoc.value.textEntrat.toLowerCase();
  estatDelJoc.value.textEntrat = entrada;

  if (entrada.length === 1 && tempsIniciParaula === 0) {
    iniciarCronometreParaula();
  }

  const paraula = paraulaActiva.value;

  // 🔹 Comprobar letra por letra solo hasta la longitud actual
  for (let i = 0; i < entrada.length; i++) {
    // Inicializamos un array de tracking si no existe
    paraula._errors = paraula._errors || [];

    // Si la letra es incorrecta y no estaba marcada antes, sumamos
    if (entrada[i] !== paraula.text[i] && !paraula._errors[i]) {
      paraula.errors++;               // errores de la palabra
      estatDelJoc.value.errorglobal++; // errores globales
      paraula._errors[i] = true;      // marcamos esta posición como error ya contada
    }
  }

  // Si la palabra se ha completado correctamente
  if (entrada === paraula.text) {
    const tempsTrigat = Date.now() - tempsIniciParaula;

    estatDelJoc.value.estadistiques.push({
      paraula: paraula.text,
      temps: tempsTrigat,
      errors: paraula.errors
    });

    paraula.estat = 'completada';
    estatDelJoc.value.indexParaulaActiva++;
    estatDelJoc.value.textEntrat = '';
    tempsIniciParaula = 0;
  }
}
</script>

<template>
  <div class="game-engine">
    <div class="paraules-container">
      <div 
        v-for="(paraula, index) in estatDelJoc.paraules" 
        :key="paraula.id"
        class="paraula"
        :class="{ 'paraula-activa': index === estatDelJoc.indexParaulaActiva }"
      >
        <span
          v-for="(lletra, idx) in paraula.text.split('')"
          :key="idx"
          :class="obtenirClasseLletra(lletra, idx)"
        >
          {{ lletra }}
        </span>
      </div>
    </div>

    <input 
      type="text" 
      class="text-input"
      v-model="estatDelJoc.textEntrat"
      @input="validarProgres"
      placeholder="Comença a escriure..."
    />

    <!-- Contador de errores globales y de palabra activa -->
    <div class="estadistiques">
      <p><strong>Errors globals:</strong> {{ estatDelJoc.errorglobal }}</p>
      <p v-if="paraulaActiva">
        <strong>Errors actuals:</strong> {{ paraulaActiva.errors }}
      </p>
    </div>
  </div>
</template>

<style scoped>
.paraula-activa {
  background-color: lightgrey;
  width: fit-content;
}
.paraula-activa .lletra-correcta {
  color: green;
}
.paraula-activa .lletra-incorrecta {
  color: red;
}
.estadistiques {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background: #f9f9f9;
  border-radius: 8px;
  width: fit-content;
  border: 1px solid #ddd;
}
.estadistiques p {
  margin: 0.2rem 0;
  color: #333;
}
</style>
