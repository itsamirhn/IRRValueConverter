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
  data () {
    return {
      amount: '',
      date: ''
    }
  }
}
</script>
