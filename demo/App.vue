<template>
  <div id="app" class="container mt-5">
    <h1 class="border-bottom pb-2 mb-4">Vue pivot table demo</h1>

    <h2 class="border-bottom pb-2 mb-4">Pivot <small>(drag & drop UI + PivotTable)</small></h2>

    <div class="mb-5">
      <pivot
        :data="asyncData"
        :fields="fields"
        :available-field-keys="availableFieldKeys"
        :row-field-keys="rowFieldKeys"
        :col-field-keys="colFieldKeys"
        :reducer="reducer"
        :default-show-settings="defaultShowSettings"
      >
      <template v-slot:values="{ value, row, col, rowtotal}">
        {{ (value[1] || 0 ) | number }}
      </template>
      <template v-slot:value="{ value, row, col, rowtotal}">
        <p v-if="rowtotal !== undefined && col.length > 0">
          {{ ((value[1] || 0 ) / (rowtotal[1] || 1 )) | percent }}
        </p>
        <p v-else>
          {{ (value[1] || 0 ) | number }}
        </p>
      </template>
      <template  v-slot:datasetLinkHeader="{ value }">
          <a :href="`${dataset_link(value)}`">{{ dataset_name(value) }}</a>
      </template>
      <template  v-slot:datasetHeader="{ value }">
          {{ dataset_name(value) }}
      </template>
      <template v-slot:computing>
        <div class="position-absolute w-100 h-100 text-center" style="z-index: 1;">
          <div class="position-sticky bg-white d-inline-block mt-5 p-3" style="top: 0;">
            <svg aria-hidden="true" data-prefix="fas" data-icon="spinner" role="img" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" class="svg-inline--fa fa-spinner fa-fw fa-pulse"><path fill="currentColor" d="M304 48c0 26.51-21.49 48-48 48s-48-21.49-48-48 21.49-48 48-48 48 21.49 48 48zm-48 368c-26.51 0-48 21.49-48 48s21.49 48 48 48 48-21.49 48-48-21.49-48-48-48zm208-208c-26.51 0-48 21.49-48 48s21.49 48 48 48 48-21.49 48-48-21.49-48-48-48zM96 256c0-26.51-21.49-48-48-48S0 229.49 0 256s21.49 48 48 48 48-21.49 48-48zm12.922 99.078c-26.51 0-48 21.49-48 48s21.49 48 48 48 48-21.49 48-48c0-26.509-21.491-48-48-48zm294.156 0c-26.51 0-48 21.49-48 48s21.49 48 48 48 48-21.49 48-48c0-26.509-21.49-48-48-48zM108.922 60.922c-26.51 0-48 21.49-48 48s21.49 48 48 48 48-21.49 48-48-21.491-48-48-48z" class=""></path></svg>
            Loading table values...
          </div>
        </div>
      </template>
      </pivot>
    </div>
  </div>
</template>

<script>
import Pivot from '../src/Pivot'
import data from './data'
import 'flag-icon-css/css/flag-icon.css'

function listSort(order){
  order = order.reduce((r, v, i) => {
    r[v] = i+1;
    return r;
  }, {})
  return (a, b) => (order[a] || Infinity) - (order[b] || Infinity)
}

data.fields.map(x => {
	x.getter = o=>o[x.key]
  x.valueFilter = true
  if (Array.isArray(x.sort)){
  	x.sort = listSort(x.sort)
  }
  if (x.key==="dataset_id"){
  	x.label = "Dataset"
    x.valueFilterSlotName = "datasetHeader"
  	x.headers = [{
      slotName: 'datasetLinkHeader',
      label: 'Dataset',
      checked: true
    }]
  }
})

export default {
  name: 'app',
  components: {Pivot},
  data: () => {
    return {
      data: Object.freeze(data),
      asyncData: [],

      // Pivot params
      fields: data.fields,
      availableFieldKeys: data.fields.map(x=> x.key),
      rowFieldKeys: data.rowFieldKeys,
      colFieldKeys: data.colFieldKeys,
      reducer: (cum, el) => {
          if (!Array.isArray(cum)) {
          	cum = [new Set(), cum]
          }
          if (!cum[0].has(el.id)) {
              cum[0].add(el.id)
              cum[1] += el.count || 1
          }
          return cum
      },
      defaultShowSettings: true,
      isDataLoading: false
    }
  },
  methods: {
    dataset_link: function(id) {
      return id
    },
    dataset_name: function(id) {
    	return "Dataset - " + id
    },
  },
  created: function() {
    this.asyncData = data.data  
  },
  filters: {
    percent: function(value) {

      return value.toLocaleString(undefined, {style: 'percent', maximumFractionDigits: 2})
    },
    number: function(value) {
      return value.toLocaleString()
    },
    capitalize: function(value) {
      return value.replace(/\b\w/g, l => l.toUpperCase())
    }
  }
}
</script>

<style lang="scss">
$enable-rounded: false;
$table-cell-padding: .5rem; // default in bs5
$table-cell-padding-sm: .25rem; // default in bs5
@import '~bootstrap/scss/bootstrap.scss';

/* Table with aria-busy attribute */
table[aria-busy='true'] {
  opacity: 0.6;
}

/* FontAwesome icons */
.svg-inline--fa.fa-fw {
  width: 1.25em;
}

.svg-inline--fa {
  display: inline-block;
  font-size: inherit;
  height: 1em;
  overflow: visible;
  vertical-align: -.125em !important;
}

.fa-pulse {
  animation: fa-spin 1s infinite steps(8);
}

@keyframes fa-spin {
  0% {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(1turn);
  }
}
</style>
