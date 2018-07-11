<template>
  <q-page padding>
    <q-page-sticky position="top" expand>
      <q-card class="full-width bg-white">
        <q-card-main>
          <div class="row gutter-md">
            <div class="col-2">
              <q-select float-label="Data Source" v-model="dataSource" :options="dataSources" class="no-margin" />
            </div>
            <div class="col-2">
              <q-select float-label="Filter" v-model="filter" :options="filters" @input="setFilterType" class="no-margin" />
            </div>
            <div class="col">
              <q-select float-label=" " v-model="operator" :options="filterType === Number ? numericOperators : stringOperators" class="no-margin" />
            </div>
            <div class="col-6">
              <q-search float-label="Argument" v-model="argument" class="no-margin" @keyup.enter="executeQuery" />
            </div>
            <div class="col-auto">
              <q-btn color="primary" label="Run" size="form-label" @click.native="executeQuery" class="no-margin" />
            </div>
          </div>
        </q-card-main>
      </q-card>
    </q-page-sticky>

    <div class="q-mt-xl">
      <q-card v-for="(cardDatum, i) in cardData" :key="i" class="q-mt-lg fit query-result">
        <q-card-title>
          {{ cardDatum.name }}
          <span slot="subtitle" v-if="cardDatum.description">{{ cardDatum.description }}</span>
        </q-card-title>
        <q-card-separator />
        <q-card-main><pre>{{ cardDatum.data.join('\n') }}</pre></q-card-main>
      </q-card>
    </div>

    <q-btn
      v-back-to-top
      round
      color="primary"
      class="fixed-bottom-right"
      style="margin: 0 15px 15px 0"
    >
      <q-icon name="keyboard_arrow_up" />
    </q-btn>
  </q-page>
</template>

<script>
import { exec } from 'child_process'

export default {
  methods: {
    executeQuery () {
      console.log(this)

      const query = `spell_query=${this.dataSource}.${this.filter}${this.operator}${this.argument}`

      exec(`simc "${query}"`, { maxBuffer: 1024 * 1024 }, (err, stdout, stderr) => {
        if (err) {
          this.$q.notify({
            message: `Spell query failed: ${err.message}`,
            type: 'negative',
            position: 'top'
          })
        }

        const queryResults = stdout.split('\r\n\r\n').slice(1, -1)
        const cardData = []

        queryResults.forEach(queryResult => {
          const lines = queryResult.split('\r\n')
          const [nameLine] = lines

          if (nameLine.length === 0) {
            return
          }

          const [, name] = nameLine.match(/Name\s+: (.+) \(id=/m)
          const cardDatum = { name, data: lines }
          const descriptionLine = lines.find(line => line.startsWith('Description'))

          if (descriptionLine) {
            [, cardDatum.description] = descriptionLine.split(': ')
          }

          cardData.push(cardDatum)
        })

        this.cardData = cardData
      })
    },

    setFilterType (value) {
      switch (value) {
        case 'name':
          this.filterType = String
          break
        case 'id':
          this.filterType = Number
          break
      }
    }
  },
  data () {
    return {
      dataSources: ['Effect', 'Spell'].map(dataSource => ({ label: dataSource, value: dataSource.toLowerCase() })),
      filters: ['ID', 'Class', 'Name'].map(filter => ({ label: filter, value: filter.toLowerCase() })),
      numericOperators: ['<', '<=', '==', '>=', '>', '!='].map(operator => ({ label: operator, value: operator })),
      stringOperators: ['==', '!=', '~', '!~'].map(operator => ({ label: operator, value: operator })),
      filterType: 'numeric',
      dataSource: 'spell',
      filter: 'class',
      operator: '==',
      argument: 'mage',
      cardData: []
    }
  }
}
</script>

<style lang="stylus" scoped>
.query-result >>> pre
  font-family "Roboto Mono"
  white-space pre-wrap
</style>
