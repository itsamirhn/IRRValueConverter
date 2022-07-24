<template>
  <v-container class="text-center">
    <v-snackbar
      v-model="snackbar"
      top
      color="red"
      elevation="24"
      multi-line
    >
      <v-layout align-center>
        <v-icon right>
          mdi-alert-circle
        </v-icon>
        <v-layout column>
          <div style="font-weight: bold; font-size: larger">
            خطا!
          </div>
          <div class="mt-1">
            {{ snackbarText }}
          </div>
        </v-layout>
      </v-layout>
    </v-snackbar>
    <v-row>
      <v-col>
        <h1>تبدیل ارزش ریال 🇮🇷</h1>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <p>با این سایت میتونین قیمت  هرچیزی در هر چند وقت گذشته رو به ارزش الانش تبدیل کنین!</p>
      </v-col>
    </v-row>
    <v-form @submit.prevent v-model="isValidForm">
      <v-row>
        <v-col cols="12" lg="6">
          <CurrencyInput
            v-model="amount"
            :rules="textFieldRules"
            :options=
              '{
                "locale": "fa-IR",
                "currency": "IRR",
                "currencyDisplay": "symbol",
                "hideCurrencySymbolOnFocus": false,
                "hideGroupingSeparatorOnFocus": false,
                "hideNegligibleDecimalDigitsOnFocus": false,
                "autoDecimalDigits": false,
                "useGrouping": true,
                "accountingSign": false
             }'
          />
        </v-col>
        <v-col cols="12" lg="6" style="display: flex; align-items: center">
          <vue-persian-datetime-picker
            v-model="date"
            :showNowBtn="false"
            :convertNumbers="true"
            inputFormat="YYYY-MM-DD"
            displayFormat="jYYYY/jMM/jDD"
            placeholder="تاریخ قیمت قدیم مثلا ۱۳۹۳/۱۰/۲۱"
            min="2012-10-12"
            :max="yesterdayDateString"
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
          <p class="mt-3" style="font-size: x-large; text-align: center">
          <span style="font-weight: bold">
            {{ currency(amount) }} ریال
          </span>
          تو تاریخ
          <span>
            {{ toPersianDigits(jalaaliToDate(this.date).format('jDD jMMMM jYYYY')) }}،
          </span>
          یعنی حدوداً
          <span style="font-weight: bold">
            {{ toPersianDigits(jalaaliToDate(this.date).fromNow()) }}،
          </span>
          معادل
          <span style="font-weight: bolder">
            {{ currency(convertedAmount) }} ریال
          </span>
          امروزه !
        </p>
        </v-container>
      </v-card>
    </v-dialog>
    <ul class="text-right mt-5" style="line-height: 1.9em">
      <li>
        این سایت چجوری کار میکنه؟
        <span class="grey--text">
          با مقایسه قیمت <span style="font-weight: bolder">دلار</span>!
        </span>
      </li>
      <li>
        یعنی محاسبه‌اش دقیق دقیق نیست؟
        <span class="grey--text">
          نه نیست چون فاکتور‌های زیادی لحاظ نمیشه و صرفا یه مقایسه نسبی ساده‌ست.
        </span>
      </li>
      <li>
        قیمت دلار از کجا میاد؟
        <span class="grey--text">
          از دیتا ذخیره شده‌ی <span style="font-weight: bolder">رایگان</span> روزهای گذشته سایت <a href="https://bonbast.com">بن‌بست</a>.
        </span>
      </li>
      <li>
        چرا بعضی تاریخ‌ها غیرفعالن؟
        <span class="grey--text">
          چون دیتایی از قیمت دلار اون روز‌ها به صورت آزاد در دسترس نیست.
        </span>
      </li>
      <li>
        هدف این سایت چیه؟
        <span class="grey--text">
          هیچ هدف خاصی نداره و صرفا از جهت  فان ساخته شده تا سال‌هایی که سپری کردیم رو بهتر یادمون بیاد :))
        </span>
      </li>
      <li>
        کد سایت اوپن‌سورسه؟
        <span class="grey--text">
          اره از این <a href="https://github.com/itsamirhn/IRRValueConverter">لینک</a> میتونی کد‌هاشو ببینی و خوشحال میشم یه <span style="font-weight: bolder">Star ⭐️</span> بدی!
        </span>
      </li>
      <li>
         چرا باید از فیلترشکن استفاده کرد؟
        <span class="grey--text">
          چون برای سرور‌ این وبسایت از سرویس‌های رایگان خارجی استفاده شده و برای دور زدن تحریم‌های این سرویس باید از این روش استفاده کرد.
        </span>
      </li>
    </ul>
  </v-container>
</template>

<script>
import moment from 'moment-jalaali';
import VuePersianDatetimePicker from 'vue-persian-datetime-picker';
import CurrencyInput from "@/components/CurrencyInput";

export default {
  name: 'IndexPage',
  components: {
    VuePersianDatetimePicker,
    CurrencyInput
  },
  data () {
    return {
      snackbar: false,
      snackbarText: 'هیچی!',
      isValidForm: false,
      isLoading: false,
      dialog: false,
      amount: null,
      convertedAmount: '',
      date: '',
      textFieldRules: [
        v => !!v || 'قیمت رو وارد کن',
      ],
      yesterdayDateString: moment().subtract(1, 'days').format('YYYY-MM-DD'),
    }
  },
  methods: {
    calculate () {
      this.isLoading = true
      this.fetchArchiveUSDPrice(this.jalaaliToGregorian(this.date)).then(async (oldPrice) => {
        const newPrice = await this.fetchLiveUSDPrice()
        const number = Number(Number(this.amount) * newPrice / oldPrice).toFixed(0)
        this.convertedAmount = number.toString()
        this.isLoading = false
        this.dialog = true
      }).catch(() => this.isLoading = false)
    },
    jalaaliToDate(jalaaliStr) {
      return moment(jalaaliStr, 'jYYYY/jMM/jDD')
    },
    jalaaliToGregorian(jalaaliStr) {
      return this.jalaaliToDate(jalaaliStr).format('YYYY-MM-DD')
    },
    toPersianDigits(str) {
      const e2p = s => s.replace(/\d/g, d => '۰۱۲۳۴۵۶۷۸۹'[d])
      return e2p(str)
    },
    currency (amount) {
      return Number(amount).toLocaleString('fa')
    },
    fetchLiveUSDPrice() {
      const request = this.fetch(process.env.live_usd_api)
      return request.then(res => res.data['usd']['sell'])
    },
    fetchArchiveUSDPrice(date) {
      const request = this.fetch(process.env.archive_usd_api, {
        timeout: 5000,
        params: {
          date: date
        }
      })
      return request.then(res => res.data['usd']['sell'])
    },
    fetch(path, config) {
      return this.$axios.get(path, config).catch((reason) => {
        if (reason.code === 'ECONNABORTED') {
          this.snackbarText = 'به دلیل تحریم‌ها، لطفا از وی‌پی‌ان استفاده کن.'
          this.snackbar = true
        } else {
          this.snackbarText = 'یه مشکلی پیش اومد. لطفا دوباره تلاش کن.'
          this.snackbar = true
        }
      })
    }
  }
}
</script>
