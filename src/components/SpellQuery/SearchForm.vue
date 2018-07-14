<template>
  <div>
    <q-card class="bg-white q-mb-xl">
      <q-card-main>
        <div
          class="row gutter-sm justify-between"
          v-if="manualMode"
        >
          <div class="col">
            <q-input
              v-model="manualQuery"
              stack-label="Manual Query"
              prefix="spell_query="
              class="no-margin"
              @keyup.enter="executeQuery"
            />
          </div>
          <div class="col-auto">
            <q-btn-group push>
              <q-btn
                push
                color="primary"
                label="Run"
                size="form-label"
                @click="executeQuery"
              />
              <q-btn
                push
                color="primary"
                label="Cancel"
                size="form-label"
                @click="manualMode = false"
              />
            </q-btn-group>
          </div>
        </div>

        <div
          class="row gutter-md no-wrap"
          v-else
        >
          <div class="col">
            <q-select
              float-label="Data Source"
              v-model="dataSource"
              :options="dataSources"
            />
          </div>
          <div class="col">
            <q-select
              float-label="Filter"
              v-model="filter"
              :options="filters"
              @input="setFilterType"
            />
          </div>
          <div class="col">
            <q-select
              float-label=" "
              v-model="operator"
              :options="filterType === Number ? numericOperators : stringOperators"
            />
          </div>
          <div class="col">
            <q-search
              float-label="Argument"
              v-model="argument"
              @keyup.enter="executeQuery"
            />
          </div>
          <div class="col-auto">
            <q-btn-group push>
              <q-btn
                push
                color="primary"
                label="Run"
                size="form-label"
                @click="executeQuery"
              />
              <q-btn
                push
                color="primary"
                label="Premades"
                size="form-label"
                @click="showPremades"
              />
              <q-btn
                push
                color="primary"
                label="Manual"
                size="form-label"
                @click="manualMode = true"
              />
            </q-btn-group>
          </div>
        </div>
      </q-card-main>
    </q-card>

    <q-dialog
      stack-buttons
      v-model="premadesDialog"
      title="Premade Queries"
      class="premades-dialog"
    >
      <template
        slot="buttons"
        slot-scope="props"
      >
        <q-btn
          v-for="(query, i) in premadeQueries"
          :key="i"
          :label="query.label"
          @click="query.handler(props.ok)"
          flat
          class="full-width"
        />
      </template>
    </q-dialog>
  </div>
</template>

<i18n>
en-us:
  spellQuery:
    spellByNameTitle: Find a spell by name
    spellByNameBody: Enter a spell name. You can enter a literal or pre-formatted name, e.g. Arcane Missiles or arcane_missiles.
</i18n>

<script>
function createSelectChoices (choices) {
  return choices.map(choice => {
    if (typeof choice === 'string') {
      return {label: choice, value: choice.toLowerCase().replace(' ', '_')}
    }

    return choice
  })
}

export default {
  name: 'SearchForm',
  data () {
    return {
      classes: createSelectChoices([
        'Death Knight',
        'Druid',
        'Hunter',
        'Mage',
        'Monk',
        'Paladin',
        'Priest',
        'Rogue',
        'Shaman',
        'Warlock',
        'Warrior'
      ]),
      dataSources: createSelectChoices(['Azerite', 'Effect', 'Spell']),
      filters: createSelectChoices(['Class', 'ID', 'Name']),
      numericOperators: createSelectChoices(['<', '<=', '==', '>=', '>', '!=']),
      stringOperators: createSelectChoices(['==', '!=', '~', '!~']),
      filterType: 'numeric',
      dataSource: 'spell',
      filter: 'class',
      operator: '==',
      argument: 'mage',
      premadesDialog: false,
      premadeQueries: [
        {label: 'Find all Azerite traits for a class', handler: this.azeriteByClass},
        {label: 'Find a spell by its name', handler: this.spellByName}
      ],
      classDialog: false,
      manualMode: false,
      manualQuery: null
    }
  },
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
        actions: this.premadeQueries
      })
    },
    showPremadesDialog () {
      this.premadesDialog = true
    },
    async classPrompt () {
      return this.$q.dialog({
        options: {
          type: 'radio',
          model: '',
          items: this.classes
        },
        cancel: true,
        preventClose: true
      })
    },
    async textPrompt (promptTitle, promptMessage) {
      return this.$q.dialog({
        title: promptTitle,
        message: promptMessage,
        prompt: {
          model: ''
        },
        cancel: true,
        preventClose: true
      })
    },
    async azeriteByClass (okFn) {
      await okFn()

      try {
        const className = await this.classPrompt()

        if (className == null || className.trim() === '') {
          return this.$q.notify({
            type: 'negative',
            position: 'top',
            message: 'A class was not selected.'
          })
        }

        this.dataSource = 'azerite'
        this.filter = 'class'
        this.operator = '=='
        this.argument = className

        this.executeQuery()
      } catch (e) {
      }
    },
    async spellByName (okFn) {
      await okFn()

      try {
        const name = await this.textPrompt(
          this.$t('spellQuery.spellByNameTitle'),
          this.$t('spellQuery.spellByNameBody')
        )

        if (name == null || name.trim().length === 0) {
          return this.$q.notify({
            type: 'negative',
            position: 'top',
            message: 'No spell name was specified.'
          })
        }

        this.dataSource = 'spell'
        this.filter = 'name'
        this.operator = '=='
        this.argument = name.replace(/[-']/, '').replace(' ', '_')

        this.executeQuery()
      } catch (e) {
      }
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
    },
    executeQuery () {
      if (this.manualMode && (this.manualQuery == null || this.manualQuery.trim().length === 0)) {
        return this.$q.notify({
          type: 'negative',
          position: 'top',
          message: 'Spell query is blank.'
        })
      }

      const query = `spell_query=${this.manualMode
        ? this.manualQuery
        : `${this.dataSource}.${this.filter}${this.operator}${this.argument}`}`

      this.$emit('execute-query', query)
    }
  }
}
</script>

<style
  lang="stylus"
  scoped
>
.premades-dialog >>> .modal-header
  text-align: center
</style>
