<template>
  <q-card>
    <q-card-section>
      <q-table
        dense
        flat
        bordered
        title="User Access Audit List"
        row-key="name"
        separator="cell"
        color="accent"
        virtual-scroll
        style="max-height: 50vh;"
        hide-bottom
        :data="data"
        :columns="columns"
        :filter="filter"
        :pagination.sync="pagination"
        :rows-per-page-options="[0]"
        :loading="data.length > 0 ? false : true"
      >
        <template v-slot:top-right>
          <!-- BUTTONS -->
          <q-btn-group
            unelevated
            class="q-mr-sm"
          >
            <q-btn
              outline
              color="accent"
              label="CSV"
              @click="exportData"
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
import { info } from 'boot/notifier'

export default {
  props: ['data'],

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
          let align, headerStyle

          switch (col) {
            default:
              align = 'left'
              headerStyle = 'text-align: left;'
              break
          }

          return {
            name: col,
            field: col,
            label: col,
            align: align,
            style: 'max-width: 300px;',
            classes: 'ellipsis',
            headerStyle: headerStyle
          }
        })

      this.columns = cols
    }
  },

  methods: {
    exportData () {
      const title = 'User Access Audit List'
      this.export(title, unparse(this.data))
    },

    export (name, data) {
      info('Downloading Data', 'Please wait')
      exportFile(`${name}.csv`, data)
    }
  }
}
</script>
