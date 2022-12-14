<template>
  <q-card>
    <q-card-section>
      <q-table
        dense
        flat
        bordered
        :title="title"
        :title-class="titleClass"
        row-key="name"
        separator="cell"
        color="accent"
        :data="data"
        :columns="columns"
        :filter="filter"
        :loading="data.length > 0 ? false : true"
        class="table-style"
        :id="title"
      >
        <template v-slot:top-right>
          <!-- BUTTONS -->
          <q-btn-group
            outline
            class="q-mr-sm"
          >
            <q-btn
              outline
              color="accent"
              label="XLSX"
              @click="exportData2"
            />
          </q-btn-group>

          <!-- SEARCH -->
          <q-input
            dense
            standout
            debounce="300"
            v-model="filter"
            placeholder="Search"
          >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </template>
      </q-table>
    </q-card-section>
  </q-card>
</template>

<script>
import { first } from 'lodash'
import { exportFile } from 'quasar'
import { unparse } from 'papaparse'
import XLSX from 'xlsx'
import { info } from 'boot/notifier'

export default {
  props: ['title', 'titleClass', 'data'],

  data () {
    return {
      columns: [],
      filter: '',
      pagination: {
        rowsPerPage: 0
      }
    }
  },

  watch: {
    data (val) {
      const cols = Object
        .keys(first(val))
        .map(col => {
          return {
            name: col,
            field: col,
            label: col,
            align: 'left',
            style: 'min-width: 150px;',
            classes: 'ellipsis',
            headerClasses: 'text-weight-bolder',
            headerStyle: 'text-align: left;'
          }
        })

      this.columns = cols
    }
  },

  methods: {
    exportData () {
      const title = `Monthly Leavers for ${this.title}`
      this.export(title, unparse(this.data))
    },

    export (name, data) {
      info('Downloading Data', 'Please wait')
      exportFile(`${name}.csv`, data)
    },

    exportData2 () {
      const title = `Monthly Leavers for ${this.title}`
      this.exportXLSX2(title, this.data)
    },

    exportXLSX2 (name, data) {
      const wb = XLSX.utils.book_new()
      const ws = XLSX.utils.json_to_sheet(data)

      XLSX.utils.book_append_sheet(wb, ws, 'Sheet 1')
      XLSX.writeFile(wb, `${name}.xlsx`)
    }
  }
}
</script>

<style lang="scss" scoped>
.table-style {
  max-height: 80vh;
}
</style>
