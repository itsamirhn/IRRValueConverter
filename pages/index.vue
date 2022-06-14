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
    <v-form>
      <v-row>
        <v-col>
          <v-text-field v-model="amount" label="قیمت" aria-label="Amount" required/>
        </v-col>
      </v-row>
      <v-row>
        <v-col>
          <v-date-picker
            v-model="date"
            :max="(new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().slice(0, 10)"
            locale="fa-IR"
            first-day-of-week="6"
            aria-label="Date"
          ></v-date-picker>
        </v-col>
      </v-row>
    </v-form>
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
      amount: '',
      date: '',
      usdPrices: {},
    }
  }
}
</script>
