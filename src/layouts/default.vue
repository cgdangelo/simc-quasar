<template>
  <q-layout view="hHh Lpr lFf">
    <q-layout-header>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          @click="leftDrawerOpen = !leftDrawerOpen"
          aria-label="Menu"
        >
          <q-icon name="menu" />
        </q-btn>

        <q-toolbar-title>
          Quasar App
          <div slot="subtitle">Running on Quasar v{{ $q.version }}</div>
        </q-toolbar-title>

        <q-select
          v-model="$i18n.locale"
          :options="availableLocales"
          color="primary"
          inverted
        />
      </q-toolbar>
    </q-layout-header>

    <q-layout-drawer
      v-model="leftDrawerOpen"
      :content-class="$q.theme === 'mat' ? 'bg-grey-2' : null"
    >
      <q-list
        no-border
        link
        inset-delimiter
      >
        <q-item to="spell_query">
          <q-item-side icon="search" />
          <q-item-main label="Spell Query" />
        </q-item>
      </q-list>
    </q-layout-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
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
      ]
    }
  },
  methods: {
    openURL
  }
}
</script>

<style>
</style>
