<script setup>
import IconoExpandir from "../iconos/IconoExpandir.vue";
import IconoContraer from "../iconos/IconoContraer.vue";
import { onMounted, ref, watch, computed } from "vue";

const instituciones = ref([]);
const inputInstitucion = ref(null);
const institucionSeleccionada = ref(null);
const institucionID = ref(null);
const subsetInstituciones = computed(() =>
  instituciones.value.filter((d) =>
    d.display_name.toLowerCase().includes(inputInstitucion.value),
  ),
);
const conjuntosInstitucion = ref([]);
const recursosConjunto = ref({});

async function seleccionarInstitucion(institucion) {
  console.log("la institucion: ", institucion);
  institucionSeleccionada.value = institucion;
  inputInstitucion.value = institucion.display_name;
  institucionID.value = institucion.id;

  const requestPackages = await fetch(
    `https://www.datos.gob.mx/api/3/action/package_search?fq=organization:${institucion.name}&rows=1000`,
  );
  const responsePackages = await requestPackages.json();
  conjuntosInstitucion.value = responsePackages.result.results;
  console.log("los conjuntos de la institución", conjuntosInstitucion.value);
  conjuntosInstitucion.value.forEach((d) => {
    recursosConjunto.value[d.title] = {
      mostrar: false,
      recursos: d["resources"],
    };
  });
}

function mostrarRecursos(conjunto) {
  let entrada = recursosConjunto.value[conjunto["title"]];
  entrada.mostrar = true;
}

function ocultarRecursos(conjunto) {
  let entrada = recursosConjunto.value[conjunto["title"]];
  entrada.mostrar = false;
}

function solicitarCSV(recurso) {
  console.log(recurso);
}

onMounted(async () => {
  let i = 0;
  let blockSize = 25;
  let next = true;
  do {
    const request = await fetch(
      `https://www.datos.gob.mx/api/3/action/organization_list?all_fields=True&offset=${i * blockSize}&limit=${blockSize}`,
    );
    const response = await request.json();
    let elements = response.result.length;
    instituciones.value = [...instituciones.value, ...response.result];
    if (elements === 0) {
      next = false;
    }

    i++;
  } while (next === true);
  instituciones.value = instituciones.value.sort(
    (a, b) => a["display_name"] - b["display_name"],
  );
});

//watch(inputInstitucion, (nv) => console.log(nv));
</script>

<template>
  <p>
    En esta sección podrás comparar el archivo que estás trabajando con
    cualquier archivo que vive en el CKAN. Esto te permitirá confirmar que las
    bases de actualización tengan la misma estructura y que no se esté tratando
    de publicar la misma base dos veces como si fueran diferentes.
  </p>

  <div>
    <label for="buscador-instituciones">Busca una institución</label>
    <input
      id="myInput"
      type="text"
      name="institución"
      v-model="inputInstitucion"
    />
    <div
      v-for="institucion in subsetInstituciones"
      v-if="instituciones.length > 0"
      class="contenedor-institucion"
      @click="seleccionarInstitucion(institucion)"
    >
      {{ institucion.display_name }}
    </div>

    A continuación, selecciona un conjunto de datos. Al seleccionarlo se
    desplegará la lista de los recursos que contiene, selecciona aquel con el
    que desees comparar la base de datos que estás revisando.
    <div v-if="conjuntosInstitucion.length > 0" class="m-y-1">
      <div v-for="conjunto of conjuntosInstitucion">
        <div
          class="p-1 p-x-2 m-0 flex flex-contenido-separado contenedor-conjunto"
        >
          <div class="columna-14">{{ conjunto["title"] }}</div>

          <button
            v-if="!recursosConjunto[conjunto.title]['mostrar']"
            class="boton-chico"
            @click="mostrarRecursos(conjunto)"
          >
            <IconoExpandir />
          </button>
          <button v-else class="boton-chico" @click="ocultarRecursos(conjunto)">
            <IconoContraer />
          </button>
        </div>
        <div v-if="recursosConjunto[conjunto.title]['mostrar']">
          <div
            v-for="recurso of recursosConjunto[conjunto.title]['recursos']"
            class="contenedor-recurso p-x-4 p-y-1 m-y-0"
            @click="solicitarCSV(recurso)"
          >
            {{ recurso.name }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.contenedor-institucion {
  border: solid 1px var(--color-secundario-5);
  border-top: none;
  padding: 8px;
}

.contenedor-conjunto {
  border: solid 1px var(--color-secundario-5);
}
.contenedor-recurso {
  background-color: var(--color-neutro-1);
  border-left: solid 1px var(--color-secundario-5);
  border-right: solid 1px var(--color-secundario-5);
  border-bottom: solid 1px var(--color-secundario-5);
}
</style>
