<template>
  <div>

    <ToolbarByMonth
      class="mb-2"
      format="MM-YYYY"
      @month="changeMonth"
      :color="toolbarColor"
      :month="$route.query.month"
    />

    <v-card>

      <v-card-text
        class="text-center"
        v-if="mappedRecordsLength === 0"
      >
        <v-icon
          size="100"
          color="gray"
        >
          assignment
        </v-icon>
        <p class="font-weight-light subheading gray--text">
          Nenhum lançamento no periodo
        </p>
      </v-card-text>

      <v-list
        two-line
        subheader
        v-else
      >

        <template v-for="(records, date, index) in mappedRecords">
          <v-subheader :key="date">{{ date }}</v-subheader>
          <RecordsListItem
            v-for="record in records"
            :key="record.id"
            :record="record"
          />
          <v-divider
            :key="`${date}-${index}`"
            v-if="showDivider(index, mappedRecords)"
          ></v-divider>
        </template>

      </v-list>

      <v-footer class="pa-2">
        <v-flex text-right>
          <h3 class="font-weight-light">
            <span>Saldo do mês:</span>
            <strong
              class="ml-5"
              :class="amountColor(totalAmount)"
            >{{ formatCurrency(totalAmount) }}</strong>
          </h3>
        </v-flex>
      </v-footer>
    </v-card>
  </div>
</template>

<script>

import moment from 'moment'
import { groupBy } from '@/utils'
import amountColorMixin from './../mixins/amount-color'
import formatCurrencyMixin from '@/mixins/format-currency'
import RecordsListItem from './RecordsListItem.vue'
import RecordsService from './../services/records-service'
import ToolbarByMonth from './ToolbarByMonth.vue'

export default {
  name: 'RecordsList',
  components: {
    RecordsListItem,
    ToolbarByMonth
  },
  mixins: [
    amountColorMixin,
    formatCurrencyMixin
  ],
  data: () => ({
    records: []
  }),
  computed: {
    mappedRecords () {
      return groupBy(this.records, 'date', (record, dateKey) => {
        return moment(record[dateKey]).format('DD/MM/YYYY')
      })
    },
    mappedRecordsLength () {
      return Object.keys(this.mappedRecords).length
    },
    totalAmount () {
      return this.records.reduce((sum, record) => sum + record.amount, 0)
    },
    toolbarColor () {
      return this.totalAmount < 0 ? 'red' : 'primary'
    }
  },
  methods: {
    changeMonth (month) {
      this.$router.push({
        path: this.$route.path,
        query: { month }
      })
      this.setRecords(month)
    },
    async setRecords (month) {
      this.records = await RecordsService.records({ month })
    },
    showDivider (index, object) {
      return index < Object.keys(object).length - 1
    }
  }
}
</script>
