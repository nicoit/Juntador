<template>
  <div class="container-fluid">
    <div class="row pb-5">
      <div class="col-6">
        <logo />
      </div>
      <div class="col-6">
        <div class="row">
          <div class="col">
            <h1 class="title">
              juntador
            </h1>
          </div>
        </div>
        <div class="row">
          <div class="col">
            <h2 class="subtitle">
              te junta los sheets
            </h2>
          </div>
        </div>
      </div>
    </div>
    <div class="row pb-5">
      <div class="col">
        <nav class="nav nav-pills nav-fill" role="tablist">
          <a
            class="nav-item nav-link active"
            data-toggle="tab"
            href="#agregador"
            role="tab"
            aria-controls="agregador"
            aria-selected="true"
            id="tabagregador"
            >Agregador</a
          >
          <a
            class="nav-item nav-link"
            data-toggle="tab"
            href="#resultador"
            role="tab"
            aria-controls="resultador"
            aria-selected="false"
            id="tabresultador"
            >Resultador</a
          >
        </nav>
      </div>
    </div>
    <div class="tab-content">
      <div id="agregador" class="tab-pane active" role="tabpanel" aria-labelledby="tabagregador">
        <form class="row pt-5 form  align-items-end">
          <div class="col">
            <div class="form-group mb-2">
              <label for="Planilla">Planilla</label>
              <input
                id="Planilla"
                v-model="sheetid"
                type="text"
                class="form-control"
              />
            </div>
          </div>
          <div class="col">
            <div class="form-group mb-2">
              <label for="Tab">Tab</label>
              <input
                id="Tab"
                v-model="gdesheet"
                type="text"
                class="form-control"
              />
            </div>
          </div>
          <div class="col">
            <button
              class="button--green btn-block mb-2"
              role="button"
              @click="getGDE()"
            >
              Traer planilla
            </button>
          </div>
        </form>
        <div class="row">
          <div class="col">
            <!-- v-for es un loop del array u objeto '(dato, indice) in array/objeto' ( :key="indice" es una buena practica )-->
            <div class="row">
              <div class="col">
                Campos a priorizar de esta tabla:
                <table class="table">
                  <tr>
                    <td
                      v-for="(gdekey, index) in gdekeys"
                      :key="index"
                      :lala="gdekey"
                    >
                      <div class="form-check form-check-inline">
                        <input
                          :id="'inlineCheckbox' + index"
                          v-model="prioritarios"
                          class="form-check-input"
                          type="checkbox"
                          :value="gdekey"
                        />
                        <label class="form-check-label" :for="'inlineCheckbox' + index">
                          {{ gdekey }}
                        </label>
                      </div>
                    </td>
                  </tr>
                </table>
              </div>
            </div>
            <div class="row">
              <div class="col">
                <table class="table table-bordered">
                  <thead>
                    <tr>
                      <!-- armo los headers en base al array que hice antes -->
                      <th
                        v-for="(gdekey, index) in gdekeys"
                        :key="index"
                        :lala="gdekey"
                      >
                            {{ gdekey }}
                      </th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(gderow, rowindex) in gde" :key="rowindex">
                      <!-- para mantener el orden de los headers de toda la tabla utilizo el array de headers que no va a cambiar su orden -->
                      <!-- por cada row, hago un 'por cada header' para usarlo de indice y asi saco el valor.
                  dato de color, gderow[gdekey]  es igual a gde[rowindex][gdekey]
                  -->
                      <td
                        v-for="(gdekey, index) in gdekeys"
                        :key="index"
                        class="index"
                      >
                        {{ gderow[gdekey] }}
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div id="resultador" class="tab-pane" role="tabpanel" aria-labelledby="tabresultador">
        <div class="row pt-5">
          <div class="col">
            <table class="table table-bordered">
              <thead>
                <tr>
                  <th
                    v-for="(resultadokey, index) in resultadokeys"
                    :key="index"
                  >
                    {{ resultadokey }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <!-- v-for es un loop del array u objeto '(dato, indice) in array/objeto' ( :key="indice" es una buena practica )-->
                <tr
                  v-for="(resultadorow, rowindex) in resultado"
                  :key="rowindex"
                >
                  <td
                    v-for="(resultadokey, index) in resultadokeys"
                    :key="index"
                    class="index"
                  >
                    {{ resultadorow[resultadokey] }}
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import Logo from '~/components/Logo.vue';

export default {
  components: {
    Logo
  },
  data() {
    return {
      sheetid: '1btCvaj4lW6RWi2qtPFZeexrQ7I8jk0J-wjht3IW9bP0',
      // en this.gde deberia a estar un array con las columnas de la tabla del sheet
      gde: {},
      gdesheet: 'GDE',
      // en this.resultadokeys esta un array con las columnas de la tabla del sheet
      gdekeys: [],
      // objeto de resultados, indices por CUIL
      resultado: {},
      // en this.resultadokeys deberia a estar un array con las columnas de la tabla de resultados
      resultadokeys: [],
      // en this.prioritarios va a estar un array con las columnas a priorizar
      prioritarios: []
    };
  },
  created() {},
  methods: {
    /*
     trae el csv, utiliza sheetid y gdesheet como datos de la sheet que se va a traer.
    */
    getGDE() {
      const vm = this;
      d3.csv(
        'https://docs.google.com/spreadsheets/d/' +
          vm.sheetid +
          '/export?format=csv' // &sheet=' + vm.gdesheet
      ).then((data) => {
        // esto esta en el den, osea que d3.csv() salio bien
        // Recorre los resultados y crea un objecto guardando el row en un indice utilizando el CUIL.
        const pp = {};
        data.forEach((row, index) => {
          pp[row.CUIL] = row;
        });
        vm.gde = pp;
        // del primer row saca los campos y los guarda en un array
        if (data[0]) vm.gdekeys = Object.keys(data[0]);
        vm.prioritarios = [];
      });
    }
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
