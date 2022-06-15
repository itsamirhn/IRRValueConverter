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
        <v-col cols="12" lg="6">
          <v-text-field v-model="amount" label="قیمت به تومان" aria-label="Amount" required :rules="textFieldRules"/>
        </v-col>
        <v-col cols="12" lg="6" style="display: flex; align-items: center">
          <vue-persian-datetime-picker
            v-model="date"
            :showNowBtn="false"
            :convertNumbers="true"
            inputFormat="YYYY-MM-DD"
            displayFormat="jYYYY-jMM-jDD"
            placeholder="تاریخ قیمت مثلا ۱۳۹۰/۱۰/۲۱"
            min="2012-10-12"
            :max="new Date(Date.now() - 24 * 60 * 60 * 1000)"
            style="width: 100%"
          >
          </vue-persian-datetime-picker>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-btn
            color="light-green"
            class="white--text"
            x-large
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
            {{ toPersianDigits(jalaaliToDate(this.date).format('dddd jMMMM jYYYY')) }}
          </span>
          یعنی حدودا
          <span style="font-weight: bold">
            {{ toPersianDigits(jalaaliToDate(this.date).fromNow()) }}
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
import moment from 'moment-jalaali';
import VuePersianDatetimePicker from 'vue-persian-datetime-picker';

export default {
  name: 'IndexPage',
  components: {
    VuePersianDatetimePicker,
  },
  data () {
    return {
      isValidForm: false,
      isLoading: false,
      dialog: false,
      amount: '',
      convertedAmount: '',
      date: '',
      textFieldRules: [
        v => !!v || 'قیمت را وارد کنید',
        v => /^\d+$/.test(this.toEnglishDigits(v)) || 'قیمت باید عددی باشد'
      ]
    }
  },
  methods: {
    calculate () {
      this.isLoading = true
      this.fetchUSDPrice(this.jalaaliToGregorian(this.date)).then(async (oldPrice) => {
        const newPrice = await this.fetchLiveUSDPrice()
        const number = Number(Number(this.toEnglishDigits(this.amount)) * newPrice / oldPrice).toFixed(0)
        this.convertedAmount = number.toString()
        this.isLoading = false
        this.dialog = true
      }).catch((err) => {
        this.isLoading = false
        console.log(err)
      })
    },
    jalaaliToDate(jalaaliStr) {
      return moment(jalaaliStr, 'jYYYY/jMM/jDD')
    },
    jalaaliToGregorian(jalaaliStr) {
      return this.jalaaliToDate(jalaaliStr).format('YYYY-MM-DD')
    },
    toEnglishDigits (str) {
      const p2e = s => s.replace(/[۰-۹]/g, d => '۰۱۲۳۴۵۶۷۸۹'.indexOf(d))
      const a2e = s => s.replace(/[٠-٩]/g, d => '٠١٢٣٤٥٦٧٨٩'.indexOf(d))
      return p2e(a2e(str))
    },
    toPersianDigits(str) {
      const e2p = s => s.replace(/\d/g, d => '۰۱۲۳۴۵۶۷۸۹'[d])
      return e2p(str)
    },
    currency (amount) {
      return Number(this.toEnglishDigits(amount)).toLocaleString('fa')
    },
    fetchLiveUSDPrice() {
      const request = this.$axios.get(process.env.usd_api)
      return request.then(res => res.data['usd']['sell'])
    },
    fetchUSDPrice(date) {
      const request = this.$axios.get(process.env.usd_api, {
        params: {
          date: date
        }
      })
      return request.then(res => res.data['usd']['sell'])
    }
  }
}
</script>
