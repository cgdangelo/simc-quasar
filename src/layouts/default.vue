<template>
  <q-layout view="hHh Lpr lFf">
    <q-layout-header>
      <q-toolbar
        class="titlebar q-px-sm"
        color="orange-6"
      >
        <q-toolbar-title>SimulationCraft</q-toolbar-title>

        <q-btn
          flat
          icon="mdi-window-minimize"
          class="text-white"
          @click="minimize"
        />

        <q-btn
          flat
          icon="mdi-window-maximize"
          class="text-white"
          @click="maximize"
        />

        <q-btn
          flat
          icon="mdi-window-close"
          class="text-white"
          @click="currentWindow.close()"
        />
      </q-toolbar>

      <q-tabs align="justify">
        <q-route-tab
          label="Welcome"
          icon="home"
          to="/"
          exact
          slot="title"
        />

        <q-route-tab
          label="Spell Query"
          icon="alarm"
          to="/spell_query"
          exact
          slot="title"
        />
      </q-tabs>
    </q-layout-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import {remote} from 'electron'
import {openURL} from 'quasar'

export default {
  name: 'LayoutDefault',
  data () {
    return {
      leftDrawerOpen: this.$q.platform.is.desktop,
      availableLocales: [
        // FIXME: Not sure what to do about lack of regional indicator support on desktop.
        {label: 'English (US)', value: 'en-us', stamp: '🇺🇸'},
        {label: 'German', value: 'de', stamp: '🇺🇸'}
      ],
      currentWindow: remote.getCurrentWindow()
    }
  },
  methods: {
    openURL,
    maximize () {
      this.currentWindow.isMaximized() ? this.currentWindow.unmaximize() : this.currentWindow.maximize()
    },
    minimize () {
      this.currentWindow.minimize()
    }
  }
}
</script>

<style
  lang="stylus"
  scoped
>
.titlebar
  -webkit-app-region drag
  -webkit-user-select none

  >>> .q-btn
    -webkit-app-region no-drag
</style>
