<template>
  <q-page padding>
    <div class="row justify-center q-col-gutter-sm">
      <div
        class="col-12"
        v-show="brandCheck('Pldt')"
      >
        <TABLE
          :title="'PLDT FTE Allocation'"
          :textColor="'text-primary'"
          :data="pldtData"
        />
      </div>

      <div
        class="col-12"
        v-show="brandCheck('Smart')"
      >
        <TABLE
          :title="'SMART FTE Allocation'"
          :textColor="'text-secondary'"
          :data="smartData"
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
  name: 'FTEAllocPage',

  components: {
    TABLE: () => import('components/report/ftealloc/table5')
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

    async fetchUserDetails () {
      try {
        const { data } = await GetRepo.UserProfile(this.$q.localStorage.getItem('userAccnt'))
        const { brand, profile, vendor } = first(data)

        this.brandList = brand.split(',').map(m => m.replace(/(^|\s)\S/g, l => l.toUpperCase()))
        this.profileType = profile
        this.vendorType = vendor === '' || vendor === null ? '%' : vendor

        this.initializeData()
      } catch (err) {
        negative('Something went wrong', '')
      }
    },

    initializeData () {
      this.brandList.forEach(brand => {
        this.fetchData(brand.toLowerCase())
      })
    },

    async fetchData (brand) {
      const { data } = await GetRepo.FTEAlloc(brand)

      this[`${brand}Data`] = data
    }
  },

  beforeMount () { this.fetchUserDetails() },

  mounted () { info('Fetching Data', 'Please wait while data loads') }
}
</script>
