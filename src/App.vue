<script setup>
import LimpiezaCSV from "./components/LimpiezaCSV.vue";
import ConfiguracionDiccionarios from "./components/ConfiguracionDiccionarios.vue";
import RevisionMetadatos from "./components/RevisionMetadatos.vue";
import SideBar from "./components/base/SideBar.vue";
import { useGlobalStore } from "./stores/global.js";

const estadoGlobal = useGlobalStore();
</script>

<template>
  <main>
    <div>
      <SideBar />
    </div>
    <div class="mesa-trabajo">
      <RevisionMetadatos
        v-if="estadoGlobal.seccionSeleccionada === 'metadatos'"
      />
      <ConfiguracionDiccionarios
        v-if="estadoGlobal.seccionSeleccionada === 'diccionario'"
      />
      <LimpiezaCSV v-if="estadoGlobal.seccionSeleccionada === 'limpieza'" />
    </div>
  </main>
</template>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100..900;1,100..900&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Bevan:ital@0;1&family=Nunito:ital,wght@0,200..1000;1,200..1000&display=swap");
body[data-perfil="predeterminada"],
:root {
  --tipografia-familia: "Nunito", sans-serif;
  --tipografia-tamanio: 16px;
  --color-neutro-7: #131415;
  --color-neutro-6: #2a2b2e;
  --color-neutro-5: #44474b;
  --color-neutro-4: #61656b;
  --color-neutro-3: #899396;
  --color-neutro-2: #c9cfcf;
  --color-neutro-1: #eaecec;
  --color-neutro-0: #ffffff;

  --color-secundario-1: #e2dcde;
  --color-secundario-2: #dad3d5;
  --color-secundario-3: #af959f;
  --color-secundario-4: #9f7c89;
  --color-secundario-5: #8e6273;
  --color-secundario-6: #7d495e;
  --color-secundario-7: #6c3048;
  --color-secundario-8: #5c263b;
  --color-secundario-9: #521f33;
  --color-secundario-10: #54162e;
  --color-secundario-11: #611232;
  --color-secundario-12: #520f2a;

  /*
  --color-primario-4: #2cb4b4ff;
  --color-primario-3: #229496;
  --color-primario-1: #fb5d01ff;
  --color-primario-2: #ea7a25;
  */
  --color-primario-3: #9a2246;
  --color-primario-4: #751a35;
  --color-primario-1: #916f27;
  --color-primario-2: #a57f2c;

  --color-error-2: #f9c8cf;
  --color-confirmacion-2: #c2f5b5;
}
body {
  margin: 0px;
  padding: 0px;
  background-color: var(--color-neutro-0);
}
.mesa-trabajo {
  position: absolute;
  left: 15vw;
  width: 85%;
  margin-top: 0px;
  padding-bottom: 40px;
  padding-top: 0px;
}

@media (max-width: 800px) {
  .mesa-trabajo {
    left: 130px;
    width: calc(100% - 130px);
  }
}
</style>
