<template>
  <q-page padding>
    <div class="q-pa-md" style="max-width: 350px">
      <q-list bordered separator>
        <q-expansion-item
          default-opened
          group="menu"
          label="Sell Cashpower"
          icon="point_of_sale"
          class="text-primary"
        >
          <div class="row q-pa-md">
            <q-input
              v-model="meter"
              filled
              dense
              type="number"
              label="Cashpower Meter Number"
              clearable
              :bg-color="meterOK ? 'blue-1' : 'grey-3'"
              class="full-width q-mb-sm"
              clear-icon="close"
            >
            </q-input>

            <div
              v-for="(cost, i) in denominations"
              :key="i"
              class="col-4 q-pa-sm"
            >
              <q-btn
                @click="amount = cost.max"
                class="full-width"
                :color="amount === cost.max ? 'green' : 'primary'"
                >{{ cost.max }}
              </q-btn>
            </div>

            <div class="col-12 q-pa-sm">
              <q-input
                borderless
                v-model="amount"
                dense
                type="number"
                :hint="saleHint()"
                placeholder="Amount from 35 - 50000"
                clearable
                input-class="text-center"
                :bg-color="amountOK ? 'blue-1' : 'grey-3'"
                clear-icon="close"
              >
                <template v-slot:after>
                  <q-icon
                    v-if="amountOK && meterOK"
                    name="send"
                    color="green"
                    @click="Sell()"
                    class="q-pl-lg q-pr-md"
                  />
                </template>
              </q-input>
            </div>

            <div class="col-12 q-pa-sm" v-if="valid">
              <q-input
                v-model="formatToken"
                borderless
                type="textarea"
                dense
                placeholder="Last Token"
                input-class="text-center q-pl-sm"
                bg-color="green-1"
                readonly
                @click="Copy(formatToken)"
              >
                <!-- <template v-slot:append>
                  <q-icon
                    :name="agent.formatForSystem ?  'queue_play_next':'perm_identity'"
                    color="primary"
                    @click="agent.formatForSystem = !agent.formatForSystem"
                    class="q-px-md"
                  />
                </template> -->
              </q-input>
            </div>
          </div>
        </q-expansion-item>

        <q-expansion-item
          group="menu"
          label="Reports"
          caption="All sales activities..."
          icon="list_alt"
          class="text-primary"
        >
          <div class="row q-pa-md">
            <div class="col-12">
              <p>Coming soon</p>
            </div>
          </div>
        </q-expansion-item>
      </q-list>
    </div>
  </q-page>
</template>

<script>
import { copyToClipboard, Notify, date } from 'quasar'

const fee = {
  tariffs: [
    {
      min: -1000000,
      max: 4,
      charge: 1000000
    },
    {
      min: 5,
      max: 35,
      charge: 3
    },
    {
      min: 26,
      max: 50,
      charge: 3
    },
    {
      min: 51,
      max: 100,
      charge: 3
    },
    {
      min: 101,
      max: 200,
      charge: 5
    },
    {
      min: 201,
      max: 300,
      charge: 5
    },
    {
      min: 301,
      max: 500,
      charge: 6
    },
    {
      min: 501,
      max: 1000,
      charge: 7
    },
    {
      min: 1001,
      max: 2000,
      charge: 10
    },
    {
      min: 2001,
      max: 5000,
      charge: 15
    },
    {
      min: 5001,
      max: 10000,
      charge: 30
    },
    {
      min: 10001,
      max: 15000,
      charge: 40
    },
    {
      min: 15001,
      max: 20000,
      charge: 48
    },
    {
      min: 20001,
      max: 25000,
      charge: 58
    },
    {
      min: 25001,
      max: 30000,
      charge: 75
    },
    {
      min: 30001,
      max: 3000000,
      charge: 750000
    }
  ],
  cost(amount) {
    let a = this.tariffs.find(
      (item) => amount <= item.max && amount >= item.min
    )
    if (!a) a = { charge: 0 }
    return a.charge
  }
}
const token = {
  meter: '',
  code: '',
  amount: 0,
  name: 'Demo'
}

export default {
  name: 'Zee',
  data() {
    return {
      amount: null,
      meter: null,
      agent: {
        lastToken: null
      },
      show: false
    }
  },

  methods: {
    async Copy(text) {
      try {
        await copyToClipboard(text)
        console.log('in copy', text)
        this.$q.notify({
          message: `Token copied. Paste in SMS and send`,
          color: 'primary'
        })
      } catch (e) {}
    },
    saleHint() {
      return this.amountOK && this.meterOK
        ? `Buy D${this.amount || '0'} for ${this.meter || 'Unknown Meter'}`
        : ''
    },

    token(len) {
      let token = `${Math.random()}${Math.random()}`
      token = token.replace(/0\./g, '').substring(1, len + 1)

      return token
    },
    showLoading() {
      this.$q.loading.show()

      // hiding in 2s
      this.timer = setTimeout(() => {
        this.$q.loading.hide()
        this.show = true
        this.timer = void 0
      }, 4000)
    },
    async Sell() {
      this.showLoading()
      const sale = token
      sale.code = this.token(this.meter.length === 7 ? 16 : 20)
      sale.unit = Number.parseInt(sale.code.substr(-1)) % 2 === 0 ? 10.13 : 13.8
      sale.amount = this.amount
      sale.kwh = (sale.amount / sale.unit).toFixed(1)
      sale.meter = this.meter
      sale.name = await this.fakeName()
      sale.time = Date.now()

      this.agent.lastToken = sale
      console.log(sale)
    },
    async fakeName() {
      const res = await fetch('https://fakerapi.it/api/v1/persons?_quantity=1')
      const data = await res.json()
      const person = data.data[0]
      // console.log('person', person)
      return `${person.firstname} ${person.lastname} `
    }
  },

  computed: {
    denominations() {
      // return items 2 to 10
      return fee.tariffs.filter((item, i) => i > 0 && i < 10)
    },
    formatToken() {
      if (!this.agent.lastToken) return 'Token Details...'
      const token = this.agent.lastToken
      const mType =
        this.agent.lastToken.unit > 11 ? 'Commercial' : 'Residential'
      const time = date.formatDate(this.agent.lastToken.time, 'dddd Do MMMM')

      const length = token.code.length / 4
      let tr = token.code.split('')
      tr = tr.map((item, i) => {
        if (i % length === 0) return '  ' + item
        return item
      })
      const code = tr.join('')
      return `CODE: ${code}
      
      ${time}
      Name: ${token.name}
      ${mType} Meter: ${token.meter}
      Amount: D${token.amount} for ${token.kwh} KWh`
    },
    responses() {
      return '' // [...this.agent.response, "[+++]", ...this.dealer.response].join(' [,] ')
    },
    amountOK() {
      const OK =
        this.amount !== null && this.amount >= 35 && this.amount < 50001
      if (!OK) this.show = false
      // console.log('amountOK', OK, this.amount, 'show', this.show)
      return OK
    },
    meterOK() {
      const OK =
        this.meter !== null &&
        (this.meter.length === 7 || this.meter.length == 11)
      if (!OK) this.show = false
      // console.log('meterOK', OK, this.meter, 'show', this.show)
      return OK
    },
    valid() {
      console.log(
        'show',
        this.show,
        'meterOK',
        this.meterOK,
        'amountOK',
        this.amountOK
      )
      return this.amountOK && this.meterOK && this.show
    }
  },

  beforeDestroy() {
    if (this.timer !== void 0) {
      clearTimeout(this.timer)
      this.$q.loading.hide()
    }
  },

  created() {
    // console.log('amount', this.amountOK, this.amount)
    // this.fakeName()
  }
}
</script>
