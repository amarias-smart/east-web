<template>
  <q-page padding>
    <div class="row justify-center q-col-gutter-md">
      <!-- PLDT -->
      <div
        class="col-12 col-sm-12"
        :class="brandList.length > 1 ? 'col-md-6' : ''"
        v-show="brandCheck('Pldt')"
      >
        <SUMMARY
          title="PLDT Weekly Summary"
          txtColor="text-primary"
          brand="pldt"
          :data="pldtSummary"
          :date="pldtDate"
          :dates="pldtDateList"
          :vendors="pldtVendors"
          :vendorDis="pldtVendors.length > 1 ? false : true"
          :sites="pldtSites"
          @vendorChange="vendorChange"
          @siteChange="siteChange"
          @dateChange="dateChange"
        />
      </div>

      <!-- SMART -->
      <div
        class="col-12 col-sm-12"
        :class="brandList.length > 1 ? 'col-md-6' : ''"
        v-show="brandCheck('Smart')"
      >
        <SUMMARY
          title="SMART Weekly Summary"
          txtColor="text-secondary"
          brand="smart"
          :data="smartSummary"
          :date="smartDate"
          :dates="smartDateList"
          :vendors="smartVendors"
          :vendorDis="smartVendors.length > 1 ? false : true"
          :sites="smartSites"
          @vendorChange="vendorChange"
          @siteChange="siteChange"
          @dateChange="dateChange"
        />
      </div>

      <!-- TABLE -->
      <div class="col-12">
        <q-card>
          <q-card-section>
            <q-btn-toggle
              unelevated
              class="q-mb-sm"
              v-model="tableBrand"
              toggle-color="accent"
              :options="tableBrands"
            />

            <TABLE
              :data="tableData"
              :loading="tableLoad"
            />
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
</template>

<script>
import GetRepo from 'src/repository/get'
import jsonata from 'jsonata'
import { first, includes, sortBy, indexOf, concat } from 'lodash'
import { info, negative } from 'boot/notifier'

export default {
  name: 'UserAccessHistory',

  components: {
    SUMMARY: () => import('components/user-access/history/summary'),
    TABLE: () => import('components/user-access/history/table')
  },

  data () {
    return {
      brandList: [],
      profileType: '',
      vendorType: '',

      smartDate: '',
      pldtDate: '',
      smartDateList: [],
      pldtDateList: [],

      smartVendor: 'All',
      pldtVendor: 'All',
      smartVendors: [],
      pldtVendors: [],

      smartSite: 'All',
      pldtSite: 'All',
      smartSites: [],
      pldtSites: [],

      smartSummary: [],
      pldtSummary: [],

      tableBrand: 'pldt',
      tableBrands: [],
      tableData: [],
      tableLoad: false
    }
  },

  computed: {
    pldtVendorSite () {
      return { vendor: this.pldtVendor, site: this.pldtSite, date: this.pldtDate }
    },

    smartVendorSite () {
      return { vendor: this.smartVendor, site: this.smartSite, date: this.smartDate }
    }
  },

  watch: {
    pldtVendorSite ({ vendor, site }, { date }) {
      if (date !== '') {
        const ven = vendor === 'All' ? '%' : vendor
        const sit = site === 'All' ? '%' : site

        this.fetchSummaryData('pldt', ven, sit)
      }
    },

    smartVendorSite ({ vendor, site }, { date }) {
      if (date !== '') {
        const ven = vendor === 'All' ? '%' : vendor
        const sit = site === 'All' ? '%' : site

        this.fetchSummaryData('smart', ven, sit)
      }
    },

    tableBrand (val) {
      this.fetchRawData()
    }
  },

  methods: {
    brandCheck (brand) {
      return this.brandList.indexOf(brand) > -1
    },

    initFetchData () {
      if (includes(this.brandList, 'Pldt')) this.fetchFilters('pldt', 'p')
      if (includes(this.brandList, 'Smart')) this.fetchFilters('smart', 's')
    },

    async fetchFilters (brand, type) {
      try {
        const { data: dates } = await GetRepo.UamDataHistoryDates(brand, type)
        this[`${brand}DateList`] = dates.map(m => m.date)
        this[`${brand}Date`] = first(dates).date

        const { data: vendors } = await GetRepo.UamDataAgentsHistoryDistinctCol(this[`${brand}Date`], brand, 'Company Name', this.vendorType)
        const { data: sites } = await GetRepo.UamDataAgentsHistoryDistinctCol(this[`${brand}Date`], brand, 'Site', this.vendorType)

        if (this.vendorType !== '%') {
          this[`${brand}Sites`] = sites.map(m => m.Site)
          this[`${brand}Vendors`] = vendors.map(m => m['Company Name'])
        } else {
          this[`${brand}Sites`] = concat('All', sites.map(m => m.Site))
          this[`${brand}Vendors`] = concat('All', vendors.map(m => m['Company Name']))
        }

        this.fetchSummaryData(brand, this.vendorType, '%')

        if (brand === 'pldt') {
          this.fetchRawData()
        }
      } catch (err) {
        negative('Something went wrong', '')
      }
    },

    async fetchSummaryData (brand, vendor, site) {
      try {
        let { data: summary } = await GetRepo.UamDataSummaryHistory(this[`${brand}Date`], brand, vendor, site)

        const tableOrder = ['ACTIVE', 'TRAINEES', 'INACTIVE', 'RESIGNED']
        const expression = jsonata(`
          [$ { Table: $ } ~> $each(function($v1, $k1){
              {
                  'Date': $distinct($v1.Date),
                  'Name': $k1,
                  'LockedFte': '',
                  'Agents': $sum($v1.Agents),
                  'Complete': $sum($v1.Complete),
                  'Score': $round(($sum($v1.Complete)/$sum($v1.Agents)) * 100, 2),
                  '_children': $v1 { Lob: $ } ~> $each(function($v2, $k2) {
                      {
                          'Name': $k2,
                          'LockedFte': '',
                          'Agents': $sum($v2.Agents),
                          'Complete': $sum($v2.Complete),
                          'Score': $round(($sum($v2.Complete)/$sum($v2.Agents)) * 100, 2),
                          '_children': $v2 { Vendor: $ } ~> $each(function($v3, $k3) {
                              {
                                  'Name': $k3,
                                  'LockedFte': $v3.LockedFte,
                                  'Agents': $sum($v3.Agents),
                                  'Complete': $sum($v3.Complete),
                                  'Score': $round(($sum($v3.Complete)/$sum($v3.Agents)) * 100, 2)
                              }
                          })
                      }
                  })
              }
          })]
        `)
        summary = expression.evaluate(sortBy(summary, obj => indexOf(tableOrder, obj.Table)))

        this[`${brand}Summary`] = summary
      } catch (err) {
        negative('Something went wrong', '')
      }
    },

    async fetchRawData () {
      this.tableLoad = true

      try {
        const brand = this.tableBrand
        const date = this[`${brand}Date`]
        const vendor = this.vendorType
        const { data } = await GetRepo.UamDataAgentsHistory(date, brand, vendor)

        this.tableData = data
        this.tableLoad = false
      } catch (err) {
        negative('Something went wrong', '')
        this.tableLoad = false
      }
    },

    vendorChange ({ brand, vendor }) { this[`${brand}Vendor`] = vendor },

    siteChange ({ brand, site }) { this[`${brand}Site`] = site },

    dateChange ({ brand, date }) { this[`${brand}Date`] = date }
  },

  async beforeMount () {
    try {
      const { data: user } = await GetRepo.UserProfile(this.$q.localStorage.getItem('userAccnt'))
      const { brand, profile, vendor } = first(user)

      this.brandList = brand.split(',').map(m => m.replace(/(^|\s)\S/g, l => l.toUpperCase()))
      this.profileType = profile
      this.vendorType = vendor === '' || vendor === null ? '%' : vendor
      this.tableBrands = this.brandList.map(m => { return { label: m.toUpperCase(), value: m.toLowerCase() } })

      this.initFetchData()
    } catch (err) {
      negative('Something went wrong', '')
    }
  },

  mounted () {
    info('Fetching Data', 'Please wait while data loads')
  }
}
</script>
