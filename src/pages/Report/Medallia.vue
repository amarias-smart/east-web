<template>
  <q-page padding>
    <div class="row q-col-gutter-md justify-center">
      <!-- PLDT -->
      <div class="col-12">
        <TABLE
          class="fit"
          :title="'PLDT Medallia'"
          :titleClass="'text-primary'"
          :data="pldtData"
          v-show="brandCheck('Pldt')"
        />
      </div>

      <!-- SMART -->
      <div class="col-12">
        <TABLE
          :title="'SMART Medallia'"
          :titleClass="'text-secondary'"
          :data="smartData"
          v-show="brandCheck('Smart')"
        />
      </div>
    </div>
  </q-page>
</template>

<script>
import GetRepo from 'src/repository/get'
import { first } from 'lodash'
import { info, negative } from 'boot/notifier'

export default {
  name: 'MedalliaReport',

  components: {
    TABLE: () => import('components/report/medallia/table.vue')
  },

  data () {
    return {
      profileType: '',
      vendorType: '',
      brandList: [],

      pldtData: [],
      smartData: []
    }
  },

  methods: {
    brandCheck (brand) {
      return this.brandList.indexOf(brand) > -1
    },

    async initializeData (brand) {
      const { data } = await GetRepo.Medallia(brand, this.vendorType)

      this[`${brand}Data`] = data
    },

    fetchData () {
      this.brandList.forEach(brand => {
        this.initializeData(brand.toLowerCase())
      })
    }
  },

  async beforeMount () {
    try {
      const { data } = await GetRepo.UserProfile(this.$q.localStorage.getItem('userAccnt'))
      const { brand, profile, vendor } = first(data)

      this.brandList = brand.split(',').map(m => m.replace(/(^|\s)\S/g, l => l.toUpperCase()))
      this.profileType = profile
      this.vendorType = vendor === '' || vendor === null ? '%' : vendor

      this.fetchData()
    } catch (err) {
      negative('Something went wrong', '')
    }
  },

  mounted () {
    info('Fetching Data', 'Please wait while data loads')
  }
}
</script>
