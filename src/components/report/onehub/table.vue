<template>
  <q-card>
    <!-- TABLE -->
    <q-card-section>
      <q-table
        dense
        flat
        bordered
        title="Onehub List"
        row-key="name"
        separator="cell"
        color="accent"
        virtual-scroll
        style="max-height: 80vh;"
        :data="data"
        :columns="columns"
        :filter="filter"
        :pagination.sync="pagination"
        :rows-per-page-options="[0]"
        :loading="data.length > 0 ? false : true"
      >
        <template v-slot:top-right>
          <!-- BUTTONS -->
          <!-- <q-btn-group class="q-mr-sm">
            <q-btn
              outline
              color="accent"
              label="CSV"
              @click="exportData"
            />
          </q-btn-group> -->

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
import { info } from 'boot/notifier'

export default {
  name: 'Table',

  props: ['data'],

  data () {
    return {
      filter: '',
      columns: [],
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
          let align

          switch (col) {
            default:
              align = 'left'
              break
          }

          return {
            name: col,
            field: col,
            label: col.toUpperCase(),
            align: align,
            style: 'max-width: 300px;',
            classes: 'ellipsis',
            headerStyle: 'text-align: left;'
          }
        })

      this.columns = cols
    }
  },

  methods: {
    exportData () {
      const title = 'Onehub List'
      this.export(title, unparse(this.data))
    },

    export (name, data) {
      info('Downloading Data', 'Please wait')
      exportFile(`${name}.csv`, data)
    }
  }
}
</script>
