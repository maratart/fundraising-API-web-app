<template>
  <div>
    <h4>{{$t('hi')}}, {{getUsername}}</h4>
    <p>
      {{$t('Thanks for your generous heart. You are changing the world for a lot of people who lost hope')}}
    </p>

    <div class="btn-group btn-group-vertical btn-block" role="group" aria-label="home-menu-items">
      <button class="btn btn-default active">{{ $t('Home') }}</button>
      <button class="btn btn-default" @click="goToAssociations">{{ $t('Charities') }}</button>
      <button class="btn btn-default" @click="goToDonations" >{{ $t('My Donations') }}</button>
      <button class="btn btn-default" @click="goToSolidarityAccount"> {{ $t('Solidarity Account') }} </button>
    </div>
    <p v-if="$store.getters.getBalance">
      {{ $t('Total Donations') }}: <span class="">&euro; {{$store.getters.getBalance}}</span>
    </p>
    <p v-else>
      <i class="fa fa-spinner fa-spin fa-fw"></i>
    </p>
  </div>
</template>

<script>

export default {
  data () {
    return {
      loadingDonationMetrics: false
    }
  },
  methods: {
    goToAssociations (e) {
      e.preventDefault()
      // this.$store.commit('setCurrentPage', 'associations')
      this.$events.emit('goToPageEvent', 'associations')
      this.$store.commit('resetMessages')
    },
    goToDonations (e) {
      e.preventDefault()
      // this.$store.commit('setCurrentPage', 'donations')
      this.$events.emit('goToPageEvent', 'donations')
      this.$store.commit('resetMessages')
    },
    goToSolidarityAccount (e) {
      e.preventDefault()
      // this.$store.commit('setCurrentPage', 'solidarity')
      this.$events.emit('goToPageEvent', 'solidarity')
      this.$store.commit('resetMessages')
    }
  },
  computed: {
    getUsername () {
      var email = this.$store.getters.getEmail
      if (!email) {
        return 'user'
      }
      var parts = email.split('@')
      if (parts > 1) {
        return parts[0]
      }
      return parts[0]
    },
    gettingDonationSum () {
      return this.loadingDonationMetrics
    },
    getDonationSum () {
      // get the donation
      var vm = this
      setTimeout(() => { vm.$events.$emit('checkDonationMeterics') }, 10000)
      return this.$store.getters.getDonationsTotal
    }
  }
}
</script>
<style scoped>
h1, h2, h3, h4, h5 {
  font-weight: bold
}
h1 {
  font-size: 1.6em
}
h2{
  font-size: 1.45em
}
h3{
  font-size: 1.3em
}
h4 {
  font-size: 1.15em
}
h5 {
  font-size: 1em
}
</style>
