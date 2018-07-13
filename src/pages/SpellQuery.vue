<template>
  <q-page padding>
    <SearchForm @execute-query="executeQuery" />

    <p
      v-if="cardData.length > 0"
      class="caption"
    >
      {{ cardData.length }} results
    </p>

    <q-card
      v-for="(cardDatum, i) in cardData"
      :key="i"
      class="query-result q-mt-md"
    >
      <q-card-title class="bg-primary text-white">
        {{ cardDatum.name }}
        <span
          slot="subtitle"
          v-if="cardDatum.description"
          class="text-white"
        >
          {{ cardDatum.description }}
        </span>
      </q-card-title>
      <q-card-separator />
      <q-card-main>
        <pre>{{ cardDatum.data.join('\n') }}</pre>
      </q-card-main>
    </q-card>

    <q-btn
      v-back-to-top
      round
      color="primary"
      size="large"
      class="fixed-bottom-right q-ma-md"
    >
      <q-icon name="keyboard_arrow_up" />
    </q-btn>
  </q-page>
</template>

<script>
import {exec} from 'child_process'
import SearchForm from '../components/SpellQuery/SearchForm'

const EOL = require('os').platform() === 'win32' ? '\r\n' : '\n'

export default {
  components: {SearchForm},
  methods: {
    executeQuery (query) {
      exec(`simc "${query}"`, {maxBuffer: 1024 * 1024}, (err, stdout) => {
        if (err) {
          this.$q.notify({
            message: `Spell query failed: ${err.message}`,
            type: 'negative',
            position: 'top'
          })
        }

        const queryResults = stdout.split(`${EOL}${EOL}`).slice(1, -1)
        const cardData = []

        queryResults.forEach(queryResult => {
          const lines = queryResult.split(EOL)
          const [nameLine] = lines

          if (nameLine.length === 0) {
            return
          }

          const [, name] = nameLine.match(/Name\s+: (.+) \(id=/m)
          const cardDatum = {name, data: lines}

          // FIXME: This is broken for multi-line descriptions.
          const descriptionLine = lines.find(line => line.startsWith('Description'))

          if (descriptionLine) {
            [, cardDatum.description] = descriptionLine.split(': ')
          }

          cardData.push(cardDatum)
        })

        this.cardData = cardData
      })
    }
  },
  data () {
    return {
      cardData: []
    }
  }
}
</script>

<style lang="stylus" scoped>
.query-results
  margin-top: 5rem

.query-result >>> pre
  font-family "Roboto Mono"
  white-space pre-wrap
</style>
