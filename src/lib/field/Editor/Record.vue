<template>
  <b-form-group :label="$t(field.label) || field.name">
    <vue-select :options="options"
                :disabled="!module"
                @search="search"
                option-value="recordID"
                option-text="label"
                :placeholder="$t('field.kind.record.suggestionPlaceholder')"
                v-model="selected">

      <b-form-text v-if="validate && errors">
        <div v-for="(error, i) in errors" :key="i">{{ error }}</div>
      </b-form-text>
    </vue-select>
  </b-form-group>
</template>
<script>
import base from './base'
import { VueSelect } from 'vue-select'
import Record from '@/lib/record'
import _ from 'lodash'

export default {
  components: {
    VueSelect,
  },

  extends: base,

  data () {
    return {
      valueRecord: {},
      records: [],
      latest: [], // set of 20 latest records for default list
      query: null,
    }
  },

  computed: {
    options () {
      return (this.query ? this.records : this.latest).map(this.convert)
    },

    module () {
      if (this.field.options.moduleID !== '0') {
        return this.$store.getters['module/getByID'](this.field.options.moduleID)
      } else {
        return undefined
      }
    },

    selected: {
      get () {
        this.findByID(this.value)
        return this.convert(this.valueRecord)
      },

      set (v) {
        let { value } = v || {}
        if (value && value !== this.value) {
          // Set selected to value
          this.value = value

          // Find selected and copy it to so we can show it
          this.valueRecord = this.records.find(r => r.recordID === value)
        }
      },
    },
  },

  watch: {
    'field.options': {
      deep: true,
      handler () {
        // We need to daly loading for abit
        _.throttle((e) => {
          this.loadLatest()
        }, 500)()
      },
    },
  },

  created () {
    this.loadLatest()
  },

  methods: {
    convert (r) {
      if (!r || !r.values) {
        return null
      }

      const value = r.recordID
      let label = value
      if (this.field.options.labelField) {
        label = r.values[this.field.options.labelField]
      }

      return { value, label }
    },

    search (query) {
      this.query = query
      const namespaceID = this.namespace.namespaceID
      const moduleID = this.field.options.moduleID

      if (moduleID && query.length > 0) {
        // Prepare query
        const filter = (this.field.options.queryFields || [this.field.options.labelField]).map(qf => {
          return `${qf} LIKE '%${query}%'`
        }).join(' OR ')

        this.$compose.recordList({ namespaceID, moduleID, filter, sort: this.sortString() }).then(({ set }) => {
          this.records = set.map(r => new Record(this.module, r))
        })
      }
    },

    loadLatest () {
      const namespaceID = this.namespace.namespaceID
      const moduleID = this.field.options.moduleID
      if (moduleID) {
        this.$compose.recordList({ namespaceID, moduleID, sort: this.sortString() }).then(({ set }) => {
          this.latest = set.map(r => new Record(this.module, r))
        })
      }
    },

    sortString () {
      return [this.field.options.labelField, 'updated_at DESC', 'created_at DESC'].filter(f => !!f).join(', ')
    },

    // Fetches record if not already present
    findByID (recordID) {
      const namespaceID = this.namespace.namespaceID
      const moduleID = this.field.options.moduleID
      if (moduleID && recordID && (this.valueRecord || {}).recordID !== recordID) {
        this.$compose.recordRead({ namespaceID, moduleID, recordID }).then(r => {
          this.valueRecord = new Record(this.module, r)
        })
      }
    },
  },
}
</script>
