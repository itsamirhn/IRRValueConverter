<template>
  <v-container class="text-center">
    <v-row>
      <v-col>
        <h1>تبدیل ارزش ریال 🇮🇷</h1>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <p>با این سایت میتونین قیمت  هرچیزی در هر چند وقت گذشته رو به ارزش الانش تبدیل کنین :))</p>
      </v-col>
    </v-row>
    <v-form @submit.prevent v-model="isValidForm">
      <v-row>
        <v-col>
          <v-text-field v-model="amount" label="قیمت به تومان" aria-label="Amount" required :rules="textFieldRules"/>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-date-picker
            v-model="date"
            full-width
            :max="(new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().slice(0, 10)"
            :allowed-dates="allowedDates"
            locale="fa-IR"
            first-day-of-week="6"
            aria-label="Date"
          ></v-date-picker>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-btn
            color="light-green"
            class="white--text"
            x-large
            block
            :disabled="date === '' || (!isValidForm)"
            :loading="isLoading"
            @click="calculate">
            تبدیل ارزش قیمت قدیم به الان
          </v-btn>
        </v-col>
      </v-row>
    </v-form>
    <v-dialog
      v-model="dialog"
      width="600"
    >
      <v-card>
        <v-container>
          <p class="mt-3" style="font-size: x-large">
          <span style="font-weight: bold">
            {{ currency(amount) }} تومن
          </span>
          تو تاریخ
          <span>
            {{ new Date(date).toLocaleDateString('fa') }}
          </span>
          یعنی حدودا
          <span style="font-weight: bold">
            {{ $moment(date, "YYYY-MM-DD").fromNow().toLocaleString() }}
          </span>
          معادل
          <span style="font-weight: bolder">
            {{ currency(convertedAmount) }} تومن
          </span>
          امروزه !
        </p>
        </v-container>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  name: 'IndexPage',
  async asyncData({ $axios }) {
    let usdPrices = {}
    if (process.server) {
      usdPrices = JSON.parse(require('fs').readFileSync('../static/usd.json', 'utf8'))
    } else {
      usdPrices = await $axios.get('/usd.json').then(res => res.data)
    }
    return { usdPrices }
  },
  data () {
    return {
      isValidForm: false,
      isLoading: false,
      dialog: false,
      dialogText: '',
      amount: '',
      convertedAmount: '',
      date: '',
      usdPrices: {},
      textFieldRules: [
        v => !!v || 'قیمت را وارد کنید',
        v => /^\d+$/.test(this.toEnglishDigits(v)) || 'قیمت باید عددی باشد'
      ]
    }
  },
  computed: {
    availableDates() {
      let allowedDates = []
      for (let date in this.usdPrices) {
        allowedDates.push(date)
      }
      allowedDates.sort()
      allowedDates.reverse()
      return allowedDates
    }
  },
  methods: {
    calculate () {
      this.isLoading = true
      const number = Number(Number(this.toEnglishDigits(this.amount)) * this.usdPrices[this.availableDates[0]]['sell'] / this.usdPrices[this.date]['sell']).toFixed(0)
      this.convertedAmount = number.toString()
      this.isLoading = false
      this.dialog = true
    },
    allowedDates (a) {
      return this.availableDates.includes(a);
    },
    toEnglishDigits (str) {
      const p2e = s => s.replace(/[۰-۹]/g, d => '۰۱۲۳۴۵۶۷۸۹'.indexOf(d))
      const a2e = s => s.replace(/[٠-٩]/g, d => '٠١٢٣٤٥٦٧٨٩'.indexOf(d))
      return p2e(a2e(str))
    },
    currency (amount) {
      return Number(amount).toLocaleString('fa')
    }
  }
}
</script>
