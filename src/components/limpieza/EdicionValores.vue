<script setup>
import { onMounted, computed, ref } from "vue";
import TablaCSV from "../base/TablaCSV.vue";
import { invoke } from "@tauri-apps/api/core";
import { useDataStore } from "../../stores/data.js";

const dataStore = useDataStore();
const esquemaColumnas = computed(() =>
  dataStore.esquema.esquemaColumnas.filter((d) => d.tipo === "Texto"),
);
const columnaSeleccionada = ref(null);
const dataCategorica = ref(null);
const isLoading = ref(false);

async function seleccionarColumna(columna_seleccionada) {
  isLoading.value = true;
  console.log(columnaSeleccionada);
  columnaSeleccionada.value = columna_seleccionada.nombre;
  dataCategorica.value = await invoke("obtener_valores_categoricos", {
    columna: columna_seleccionada.nombre,
  });
  console.log("Aqui está la prueba: ", dataCategorica.value);
  isLoading.value = false;
}
onMounted(async () => {
  if (esquemaColumnas.value) {
    await seleccionarColumna(esquemaColumnas.value[0]);
  }
});
</script>
<template>
  <p>
    La herramienta permite analizar las columnas textuales que codifican
    categorías y modificar sus valores para homologarlos. En la tabla del lado
    izquierdo, da clic en la columna con la que deseas trabajar y edita los
    valores incorrectos en la tabla de la derecha.
  </p>
  <div class="flex" v-if="columnaSeleccionada">
    <div class="contenedor-tabla tabla-columnas" id="tabla-columnas ">
      <table class="columna-6">
        <thead class="header-tabla">
          <tr>
            <th>Columna</th>
            <th>Valores diferentes</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="columna in esquemaColumnas"
            :class="`fila-${columna.nombre} ${columna.nombre === columnaSeleccionada ? 'is-selected' : null}`"
            class="selector-tabla"
            @click="seleccionarColumna(columna)"
          >
            <td>{{ columna.nombre }}</td>
            <td>{{ columna.valores_unicos }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="contenedor-tabla tabla-categorias" id="tabla-categorias">
      <table class="columna-10">
        <thead class="header-tabla">
          <tr>
            <th>Original</th>
            <th>Nuevo (editable)</th>
            <th>Repeticiones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="fila in dataCategorica">
            <td>{{ fila[columnaSeleccionada] }}</td>
            <td>
              <input
                type="text"
                :id="`fila-${dataCategorica.indexOf(fila)}`"
                :name="`fila-${dataCategorica.indexOf(fila)}`"
                :value="fila[columnaSeleccionada]"
              />
            </td>
            <td>{{ fila["count"] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="flex flex-contenido-final columna-16">
      <button class="boton-primario boton-chico">Promover Cambios</button>
    </div>
  </div>

  <TablaCSV />

  <h2>Editor de valores</h2>
  <p>Da click en la columna con la que deseas trabajar:</p>
</template>
<style scoped>
.tabla-columnas {
  max-width: 30%;
}
.tabla-categorias {
  max-width: 65%;
}
.contenedor-tabla {
  max-height: 50vh;
  overflow-y: auto;
  table {
    display: inline-block;

    th {
      position: sticky;
      top: 0;
      z-index: 1;
      background-color: var(--color-neutro-1);
    }

    td {
      overflow-wrap: break-word;
      white-space: normal;
    }
  }
}
.selector-tabla {
  cursor: pointer;
}

.is-selected {
  border: 2px solid var(--color-primario-3);
}
</style>
