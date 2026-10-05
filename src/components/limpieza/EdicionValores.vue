<script setup>
import * as d3 from "d3";
import TablaCSV from "../base/TablaCSV.vue";
import { onMounted, computed, ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import { useDataStore } from "../../stores/data.js";

const dataStore = useDataStore();
const esquemaColumnas = computed(() =>
  dataStore.esquema.esquemaColumnas.filter((d) => d.tipo === "Texto"),
);
const columnaSeleccionada = ref(null);
const dataCategorica = ref(null);
const datum = ref();
const isLoading = ref(false);

async function seleccionarColumna(columna_seleccionada) {
  isLoading.value = true;
  columnaSeleccionada.value = columna_seleccionada.nombre;
  dataCategorica.value = await invoke("obtener_valores_categoricos", {
    columna: columna_seleccionada.nombre,
  });
  datum.value = dataCategorica.value.map((d) => {
    let entrada = {
      Original: d[columnaSeleccionada.value],
      Nuevo: d[columnaSeleccionada.value],
      Repeticiones: d.count,
    };
    return entrada;
  });
  isLoading.value = false;
}

async function promoverCambios() {
  for (let i in datum.value) {
    if (datum.value[i]["Original"] !== datum.value[i]["Nuevo"]) {
      const lista = [
        [
          columnaSeleccionada.value,
          datum.value[i]["Original"],
          datum.value[i]["Nuevo"],
        ],
      ];
      await invoke("actualizar_categorias", {
        cols: lista,
      });
    }
  }
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
            <th v-for="columna in Object.keys(datum[0])">{{ columna }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="fila in datum">
            <td>{{ fila["Original"] }}</td>
            <td>
              <input
                type="text"
                :id="`fila-${dataCategorica.indexOf(fila)}`"
                :name="`fila-${dataCategorica.indexOf(fila)}`"
                :value="fila['Nuevo']"
                v-model="fila['Nuevo']"
              />
            </td>
            <td>{{ fila["Repeticiones"] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="flex flex-contenido-final columna-16">
      <button class="boton-primario boton-chico" @click="promoverCambios">
        Promover Cambios
      </button>
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
