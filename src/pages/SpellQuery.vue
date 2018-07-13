<template>
  <q-page padding>
    <q-card class="bg-white">
      <q-card-main>
        <div class="row gutter-md no-wrap">
          <div class="col">
            <q-select float-label="Data Source" v-model="dataSource" :options="dataSources" />
          </div>
          <div class="col">
            <q-select float-label="Filter" v-model="filter" :options="filters" @input="setFilterType" />
          </div>
          <div class="col">
            <q-select float-label=" " v-model="operator" :options="filterType === Number ? numericOperators : stringOperators" />
          </div>
          <div class="col">
            <q-search float-label="Argument" v-model="argument" @keyup.enter="executeQuery" />
          </div>
          <div class="col-auto">
            <q-btn-group >
              <q-btn outline color="primary" label="Run" size="form-label" @click="executeQuery" />
              <q-btn outline color="primary" label="Premades" size="form-label" @click="showPremades" />
            </q-btn-group>
          </div>
        </div>
      </q-card-main>
    </q-card>

    <q-dialog stack-buttons v-model="premadesDialog" title="Premade Queries">
      <template slot="buttons" slot-scope="props">
        <q-btn outline label="List all Azerite traits for a class" />
        <q-btn outline label="List all Azerite traits for a class" />
        <q-btn outline label="List all Azerite traits for a class" />
        <q-btn outline label="List all Azerite traits for a class" />
        <q-btn outline label="List all Azerite traits for a class" />
      </template>
    </q-dialog>

    <q-card v-for="(cardDatum, i) in cardData" :key="i" class="query-result q-mt-md">
      <q-card-title class="bg-primary text-white">
        {{ cardDatum.name }}
        <span slot="subtitle" v-if="cardDatum.description" class="text-white">{{ cardDatum.description }}</span>
      </q-card-title>
      <q-card-separator />
      <q-card-main><pre>{{ cardDatum.data.join('\n') }}</pre></q-card-main>
    </q-card>

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
    showPremades () {
      if (this.$q.platform.is.mobile) {
        this.showPremadesSheet()
      } else {
        this.showPremadesDialog()
      }
    },
    showPremadesSheet () {
      this.$q.actionSheet({
        title: 'Premade Queries',
        actions: [
          { label: 'Find a spell by its id' },
          { label: 'Find a spell by its name' },
          { label: 'Find all spells for a class' }
        ]
      })
    },
    showPremadesDialog () { this.premadesDialog = true },
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
      premadesDialog: false,
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
