<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <hr>
    <input type="tel" v-model="number" placeholder="Enter a phone number. (e.g. 111-111-1111)">
    <div class="btns">
      <button class="btn" @click="() => getNumber(false)">Get Sequential Phone Number</button>
      <button class="btn" @click="() => getNumber(true)">Get Random Phone Number</button>
      <button class="btn" @click="reserveSpecific">Reserve Specified</button>
      <button class="btn" @click="check">Check Status</button>
      <button class="btn" @click="release">Release</button>
    </div>

    <div class="result" :class="hasError ? 'error' : ''"> {{ feedback }} </div>
  </div>
</template>

<script>

const BASE_URL = '/v1/number'

export default {
  data: () => ({
    number: '',
    feedback: ''
  }),
  computed: {
    numberInInt () {
      return Number(this.number.split('-').join(''))
    }
  },
  methods: {
    cleanup () {
      this.feedback = ''
      this.hasError = false
    },
    async reserveSpecific () {
      this.validateInputFormat()

      if (!this.hasError) {
        const options = {
          method: 'POST',
          cache: 'no-cache',
          headers: { 'Content-Type': 'application/json' }
        }
        const res = await fetch(`${BASE_URL}/${this.numberInInt}`, options)
          .then(res => res.json())

        if (res.success) {
          this.cleanup()
          this.number = res.number
          this.feedback = res.note || 'You reserved a specified number!'
        }
      }
    },
    async check () {
      this.validateInputFormat()

      if (!this.hasError) {
        const options = {
          method: 'GET',
          cache: 'no-cache',
          headers: { 'Content-Type': 'application/json' }
        }
        const res = await fetch(`${BASE_URL}/${this.numberInInt}`, options)
          .then(res => res.json())

        this.cleanup()
        this.feedback = res.available
          ? 'The number is available'
          : 'The number is unavailable'
      }
    },
    async release () {
      this.validateInputFormat()

      if (!this.hasError) {
        const options = {
          method: 'DELETE',
          cache: 'no-cache',
          headers: { 'Content-Type': 'application/json' }
        }
        const res = await fetch(`${BASE_URL}/${this.numberInInt}`, options)
          .then(res => res.json())

        this.cleanup()
        this.feedback = res.reason || 'This number has been released'
      }
    },
    async getNumber (isRandom = false) {
      const options = {
        method: 'POST',
        cache: 'no-cache',
        headers: { 'Content-Type': 'application/json' },
        body: isRandom ? JSON.stringify({ random: true }) : undefined
      }
      const res = await fetch(`${BASE_URL}`, options)
        .then(res => res.json())

      if (res.success) {
        this.cleanup()
        this.number = res.number
        this.feedback = 'You reserved a new number!'
      }
    },

    validateInputFormat () {
      if (this.number) {
        if (/^\d{3}-\d{3}-\d{4}$/.test(this.number)) {
          this.cleanup()
        } else {
          this.feedback = 'Invalid input format. Please enter a format like 222-222-2222'
          this.hasError = true
        }
      }

    }
  },
  name: 'app'
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

hr {
  margin: 2rem 0;
}

.btns {
  margin-top: 2rem;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn + .btn {
  margin-left: 1rem;
}

.error {
  color: red;
}

input {
  width: 360px;
  height: 32px;
  padding-left: 1rem;
  font-size: 1.1em;
}
</style>
