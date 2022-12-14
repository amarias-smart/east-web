<template>
  <q-toolbar :class="$q.dark.isActive ? 'bg-grey-10' : 'bg-grey-12'">
    <!-- DRAWER BTN -->
    <q-btn
      flat
      dense
      round
      aria-label="Menu"
      icon="mdi-chevron-left"
      :color="$q.dark.isActive ? 'grey-5' : 'grey-8'"
      :class="openDrawer ? 'rotate-0' : 'rotate-180'"
      style="transition: all ease 0.3s;"
      @click="triggerDrawer"
    />

    <!-- TITLE / LOGO -->
    <q-toolbar-title>
      <div>
        <q-img
          v-if="$q.dark.isActive"
          src="~assets/logo/east_light.png"
          style="width: 10%;"
        />
        <q-img
          v-else
          src="~assets/logo/east_dark.png"
          style="width: 10%;"
        />
      </div>
    </q-toolbar-title>

    <div
      class="q-mr-md"
      :class="$q.dark.isActive ? 'text-grey-5' : 'text-grey-8'"
    >{{ profileType }}</div>

    <q-separator
      inset
      vertical
      spaced
    />

    <!-- DROPDOWN TOOLS -->
    <q-btn-dropdown
      flat
      stretch
      label="MISC"
      :color="$q.dark.isActive ? 'grey-5' : 'grey-8'"
    >
      <q-list dense>
        <!-- LOGOUT -->
        <q-item
          clickable
          v-close-popup
          @click="logOut"
        >
          <q-item-section thumbnail>
            <q-avatar
              icon="mdi-power"
              text-color="deep-orange"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label>Logout</q-item-label>
          </q-item-section>
        </q-item>

        <!-- SETTINGS -->
        <q-item
          clickable
          :class="settings === true ? '' : 'hidden'"
          :to="{ name: 'settings' }"
        >
          <q-item-section thumbnail>
            <q-avatar
              icon="mdi-cog"
              text-color="teal"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label>Settings</q-item-label>
          </q-item-section>
        </q-item>

        <q-separator />

        <!-- INVERT COLOR -->
        <q-item
          clickable
          v-close-popup
          @click="$q.dark.toggle()"
        >
          <q-item-section thumbnail>
            <q-avatar
              icon="mdi-invert-colors"
              :text-color="$q.dark.isActive ? 'grey-1' : 'dark'"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label>Invert</q-item-label>
          </q-item-section>
        </q-item>

        <!-- FULLSCREEN -->
        <q-item
          clickable
          v-close-popup
          @click="$q.fullscreen.toggle()"
        >
          <q-item-section thumbnail>
            <q-avatar
              :icon="$q.fullscreen.isActive ? 'fullscreen_exit' : 'fullscreen'"
              text-color="purple"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label v-if="$q.fullscreen.isActive">Unfocus</q-item-label>
            <q-item-label v-else>Focus</q-item-label>
          </q-item-section>
        </q-item>

        <q-separator />

        <!-- ESPORT -->
        <q-item
          clickable
          v-close-popup
          @click="openESPort"
        >
          <q-item-section thumbnail>
            <q-avatar
              icon="mdi-anchor"
              text-color="blue"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label>ESPort</q-item-label>
          </q-item-section>
        </q-item>

        <!-- RELEASE NOTES -->
        <q-item
          clickable
          v-close-popup
          @click="showReleaseNotes"
          :style="releasenotes ? '' : 'display: none;'"
        >
          <q-item-section thumbnail>
            <q-avatar
              icon="mdi-clipboard-text-outline"
              text-color="orange"
            />
          </q-item-section>
          <q-item-section>
            <q-item-label>Releases</q-item-label>
          </q-item-section>
        </q-item>
      </q-list>
    </q-btn-dropdown>

    <!-- PLDT SMART LOGO -->
    <q-img
      src="~assets/logo/pldtsmart.png"
      style="width: 15%;"
    />
  </q-toolbar>
</template>

<script>
import GET from 'src/repository/get'
import { first } from 'lodash'
import { mapState, mapActions } from 'vuex'
import { openURL } from 'quasar'

export default {
  computed: mapState('data', ['openDrawer']),

  data () {
    return {
      profileType: '',
      settings: false,
      releasenotes: false
    }
  },

  methods: {
    ...mapActions('data', ['SET_OPENDRAWER', 'SET_ALLOW']),

    triggerDrawer () { this.SET_OPENDRAWER(!this.openDrawer) },

    logOut () {
      this.$q.loading.show()

      setTimeout(() => {
        this.SET_ALLOW(false)
        this.$q.loading.hide()
        this.$router.push({ name: 'login' })
      }, 2500)
    },

    openESPort () { openURL('http://10.122.8.191/esport') },

    showReleaseNotes () {
      this.$q.dialog({
        component: () => import('components/layout/version-logs'),
        parent: this
      })
        .onOk(() => (console.log('OK')))
        .onCancel(() => (console.log('Cancel')))
        .onDismiss(() => (console.log('Called on OK or Cancel')))
    }
  },

  async beforeMount () {
    try {
      const { data } = await GET.UserProfile(this.$q.localStorage.getItem('userAccnt'))
      const { profile, settings, releasenotes } = first(data)

      this.profileType = profile
      this.settings = settings
      this.releasenotes = releasenotes
    } catch (err) {
      console.log(err)
    }
  }
}
</script>
