<template>
  <div class="">
    <div class="other">
      <div :dir='currentLangDirection' id='wrapper' v-bind:class='getWidthClass'>
        <div class='langs btn-group btn-group-xs' role='group'>
          <button type='button' @click="setLang('en')" :class=" lang === 'en' || lang === '' ? 'btn btn-success' : 'btn btn-default'">English</button>
          <button type='button' @click="setLang('es')" :class=" lang === 'es' ? 'btn btn-success' : 'btn btn-default'">Español</button>
          <button type='button' @click="setLang('fr')" :class=" lang === 'fr' ? 'btn btn-success' : 'btn btn-default'">Français</button>
          <button type='button' @click="setLang('ar')" :class=" lang === 'ar' ? 'btn btn-success' : 'btn btn-default'">العربية</button>
          <span class='hidden'>{{currentState}}</span>
        </div>


        <div v-if="$store.getters.getShowSlide">
          <slide-page></slide-page>
        </div>

        <div class='content' v-else>
          <div class='loading' v-if="$store.getters.getLoading">
            <h1><i class='fa fa-spinner fa-spin fa-fw'></i> {{$t('Loading')}}...</h1>
          </div>

          <div v-else >
            <message-items></message-items>
            <div class='top-container'>
              <div class='top-menu'>
                <button v-if="($store.getters.getCurrentPage != 'login' && $store.getters.getCurrentPage != '' && $store.getters.getCurrentPage != 'home' && $store.getters.getCurrentState != '' && $store.getters.getCurrentPage != 'signup')" class='btn btn-plain btn-back' @click='goToPrevPage'>
                  <i v-if="langDirection == 'rtl'" class='fa fa-angle-right fa-fw'></i>
                  <i v-else class='fa fa-angle-left fa-fw'></i>
                </button>
              </div>
            </div>

            <div class=''>
              <div class='login-area' v-if="$store.getters.getCurrentState == 'login' || $store.getters.getCurrentState == ''">
                <div  v-if="$store.getters.getCurrentPage == 'login' || $store.getters.getCurrentPage == ''">
                  <login-form></login-form>
                </div>
                <div v-if="$store.getters.getCurrentPage == 'signup'">
                  <signup-form></signup-form>
                </div>
                <div v-if="$store.getters.getCurrentPage == 'signupPOS'">
                  <signup-pos-form></signup-pos-form>
                </div>
                <div v-if="$store.getters.getCurrentPage == 'share'">
                  <share-page></share-page>
                </div>

              </div>

              <div class='loggedin-area' v-else >
                <div class='logout-area'>
                  <span class='hidden'>{{getMyBalance}}</span>
                  <logout-button></logout-button>
                </div>

                <div class=''>
                  <div v-if="$store.getters.getCurrentPage == 'home' || $store.getters.getCurrentPage == ''" >
                    <home-page></home-page>
                  </div>

                  <div v-if="$store.getters.getCurrentPage == 'share'">
                    <share-page></share-page>
                  </div>

                  <div v-if="$store.getters.getCurrentPage == 'associations'">
                    <associations-page></associations-page>
                  </div>

                  <div v-if="$store.getters.getCurrentPage == 'asso_details'">
                    <association-page></association-page>
                  </div>

                  <div v-if="$store.getters.getCurrentPage == 'donations'">
                    <donations-page></donations-page>
                  </div>

                  <div v-if="$store.getters.getCurrentPage == 'solidarity'">
                    <solidarity-account-page></solidarity-account-page>
                  </div>

                  <div v-if="$store.getters.getCurrentPage == 'settings'">
                    <settings-page></settings-page>
                  </div>
                </div>

              </div>

            </div>

          </div>

          <div class='bottom-menu'>
            <label :class="$store.getters.getAppMode == 'test' ? 'test-mode version': 'version'">
              <span @click="updateVNCC">Version: {{$store.getters.getVersion}}</span>
              <span style="color: #EEE">{{$store.getters.getVNCC}}</span>
              <a href="#" @click="$store.commit('setShowSlides', true); return false"><i class='fa fa-question-circle-o fa-fw'></i></a>
              <a target='_blank' :href='helpLink'>
                {{$t('Help')}}
              </a>
            </label>
          </div>
        </div>


      </div>
    </div>
  </div>
</template>

<script>
import ga from 'vue-ga'
import VueRouter from 'vue-router'

const router = new VueRouter()
ga(router, 'UA-86334488-3')

import Home from './components/Home.vue'
import Logout from './components/Logout.vue'
import Login from './components/Login.vue'
import Signup from './components/Signup.vue'
import SignupPOS from './components/SignupPOS.vue'
import Messages from './components/Messages.vue'
import Share from './components/Share.vue'
import Associations from './components/Associations.vue'
import Association from './components/Association.vue'
import Donations from './components/Donations.vue'
import SolidarityAccount from './components/SolidarityAccount.vue'
import Settings from './components/Settings.vue'
import Slides from './components/Slides.vue'

import * as urls from './api_variables'
import axios from 'axios'
var jwtToken = localStorage.getItem('user_token')

const http = axios.create({
  headers: { 'Authorization': 'Bearer ' + jwtToken }
})

export default {
  mounted () {
    /** ** ** ** ** ** *** *** IVENTS *** *** ** ** ** ** ** ** ** ** **/
    this.$events.listen('sendLoginEvent', eventData => {
      console.log('sendLoginEvent')
      this.sendLogin(eventData)
    })

    this.$events.listen('sendEvent', eventData => {
      console.log('got sendEvent')
      console.log(eventData)
      // eventData = { type: 'type', action: 'action', label: '', value: '' }
      ga('send', 'event', eventData.type, eventData.action, eventData.label, eventData.value)
    })

    this.$events.listen('goToPageEvent', pageName => {
      console.log('got goToPageEvent')
      var slash = pageName.indexOf('/')
      if (slash > 0) {
        pageName = pageName.slice(0, slash)
      }
      this.$store.commit('setCurrentPage', pageName)
      this.$events.emit('pageChangedEvent', pageName)
    })

    ga(collect => {
      // when hash changes
      this.$events.listen('pageChangedEvent', (name) => {
        console.log('visitng ' + name)
        collect(name)
      })
    }, 'UA-86334488-3')

    this.$events.listen('skipSlideEvent', eventData => {
      console.log('got skipSlideEvent')
      this.$store.commit('setShowSlides', false)
    })

    this.$events.listen('logoutEvent', eventData => {
      console.log('LOGING OUT EVENT')
      console.log(eventData)
      this.$store.commit('logout')
      this.$store.commit('setCurrentState', 'login')
      // this.$store.commit('setCurrentPage', 'login')
      this.$events.emit('goToPageEvent', 'login')
      this.$store.commit('resetMessages')
      this.$store.commit('resetAssoList')
      // localStorage.removeItem('country')
      // localStorage.removeItem('user_token')
      // localStorage.removeItem('rememberMe')
      localStorage.clear()
      this.$store.commit('setAPI', 'mhs')
    })

    this.$events.listen('checkDonationMeterics', eventData => {
      this.getDonationMetrics()
    })

    this.$events.listen('loginEvent', eventData => {
      console.log('LOGIN EVENT')
      console.log(eventData)
      localStorage.setItem('user_email', eventData.user)
      localStorage.setItem('ss', eventData.ss)
      localStorage.setItem('user_token', eventData.token)
      this.$store.commit('setToken', eventData.token)
      localStorage.setItem('rememberMe', eventData.rememberMe)
      this.$store.commit('setCurrentState', 'loggedin')
      // this.$store.commit('setCurrentPage', 'home')
      this.$events.emit('goToPageEvent', 'home')
    })

    this.$events.listen('acountUpdate', eventData => {
      console.log('acountUpdate EVENT')
      if (!this.$store.getters.getLogin) {
        console.log('not loggedin, returning')
        return
      }
      var vm = this
      let url = urls.API_URL.CurrentUrl + urls.WALLET_BALANCE_URL
      console.log('http')
      console.log(http)
      axios({
        url: url,
        headers: { 'Authorization': 'Bearer ' + jwtToken }
      }).then(resp => {
        if (!resp.data) {
          console.log('no response data')
          return
        }
        console.log('response data')
        console.log(resp.data)
        if (resp.data.balance) {
          console.log('no balance data')
          vm.$store.commit('setCurrency', resp.data.balance.Currency)
          vm.$store.commit('setBalance', resp.data.balance.Amount)
        }
      }).catch(err => {
        console.log('axios failed')
        console.log(err.data)
      })
    })
  },
  beforeCreate () {
    console.log('beforeCreate')

    var vm = this
    setTimeout(() => { vm.$store.commit('resetMessages') }, 5000)

    var rememberMe = localStorage.getItem('rememberMe')

    if (rememberMe == null) {
      this.$store.commit('setCurrentState', 'login')
      // this.$store.commit('setCurrentPage', 'login')
      this.$events.emit('goToPageEvent', 'login')
      return 'login'
    }
    if (rememberMe !== null) {
      // read the saved token from localStorage
      var jwtToken = localStorage.getItem('user_token')
      var email = localStorage.getItem('user_email')
      var ss = localStorage.getItem('ss')
      // if found user is loggedin
      if (jwtToken !== null) {
        var loginData = {mail: email, password: ss}
        localStorage.clear()
        this.$events.emit('sendLoginEvent', loginData)
        // // check if the currentState is empty
        // if (this.$store.getters.getCurrentState === '') {
        //   if (email) {
        //     this.$store.commit('updateEmail', email)
        //     console.log('email: ' + email)
        //   } else {
        //     this.$store.commit('updateEmail', 'User')
        //   }
        //   this.$store.commit('setCurrentState', 'loggedin')
        //   this.$store.commit('setToken', jwtToken)
        //   return 'loggedin'
        // }
      } else { // no token means not signed in
        this.$store.commit('setCurrentState', 'login')
        localStorage.setItem('country_code', 'ES')
        return 'login'
      }

      return this.$store.getters.getCurrentState
    }
  },
  beforeMount () {
    this.$events.listen('changeLanguage', eventData => {
      console.log('changeLanguage')
      var lang = String(eventData).toLowerCase()
      if (lang === null || lang === 'null') {
        return
      }
      this.setLang(lang)
      console.log('changedLanguage to: ' + lang)
      this.$events.emit('languageChanged', lang)
    })

    var langFromUrl = this.getQueryParam('lang')
    this.$events.emit('changeLanguage', langFromUrl)
    console.log('langFromUrl')
    console.log(langFromUrl)
    this.$events.listen('testEvent', eventData => {
      console.log('testEvent')
      console.log(eventData)
    })
  },
  beforeUpdate () {

  },
  data () {
    return {
      langDirection: '',
      helpLink: 'https://github.com/YoQuieroAyudar/fundraising-API-web-app/wiki/Help',
      lang: 'en',
      showSlide: true
    }
  },
  methods: {
    getQueryParam (n) {
      var half = location.search.split(n + '=')[1]
      return half !== undefined ? decodeURIComponent(half.split('&')[0]) : null
    },
    setMessage (response) {
      if (!response) {
        return
      }

      var vm = this
      setTimeout(() => { vm.$store.commit('resetMessages') }, 5000)

      if (response.errors) {
        this.$store.commit('setErrors', response.errors)
      } else if (response.error) {
        this.$store.commit('setError', response.error)
      } else if (response.warning) {
        this.$store.commit('setWarning', response.warning)
      } else if (response.info) {
        this.$store.commit('setInfo', response.info)
      } else if (response.success) {
        this.$store.commit('setSuccess', response.success)
      }

      this.$store.commit('setLoading', false)
    },
    sendLogin (loginData) {
      var countryDb = localStorage.getItem('country_db')

      if (this.$store.getters.getAppMode !== 'test') {
        if (countryDb === null) {
          this.$store.commit('setAPI', 'mhs')
        } else {
          this.$store.commit('setAPI', countryDb)
        }
      }
      this.$store.commit('setLoading', true)
      let country = localStorage.getItem('country')
      if (!country) {
        country = {name: 'Spain', db: 'mhs', code: 'ES'}
        localStorage.setItem('country', JSON.stringify(country))
      }
      var creds = loginData
      // var setMyToken = this.setUserToken
      var setReponseMessage = this.setMessage
      this.$store.commit('resetMessages')
      let url = urls.API_URL.CurrentUrl + urls.LOGIN_URL
      var vm = this
      this.$http.post(url, creds)
        .then((resp) => {
          // return the success code
          var data = {}

          if (resp.status === 200) {
            if (resp.data) {
              data = resp.data
              if (data.token) {
                localStorage.setItem('user_token', data.token)
                setReponseMessage({'success': 'Login successfully!'})
                vm.$events.$emit('loginEvent', {token: data.token, user: creds.mail, ss: creds.password, rememberMe: vm.rememberMe})
                vm.$store.commit('setToken', data.token)
                vm.$events.$emit('acountUpdate', {})
              }
            }
          }
        }, (err) => {
          setReponseMessage(err.data)

          console.log('Error')
          console.log(err)
        })
    },
    updateVNCC () {
      this.$store.commit('incrementVNCC')
    },
    getDonationMetrics () {
      if ((this.$store.getters.getCurrentPage !== 'home') && (this.$store.getters.getCurrentPage !== '')) {
        return
      }
      var vm = this
      let url = urls.API_URL.CurrentUrl + urls.DONATION_URL

      http.get(url).then(resp => {
        console.log('call success')
        console.log(resp.data)
        if (resp.data) {
          if (resp.data.total_donations) {
            vm.$store.commit('setDonationTotal', resp.data)
            vm.loadingDonationMetrics = true
          }
        }
      }).catch(err => {
        console.log('call error')
        console.log(err.status + ': ' + err.statusText)
      })
      // this.$http.get(url).then(resp => {
      //   console.log('call success')
      //   console.log(resp.data)
      //   if (resp.data) {
      //     if (resp.data.total_donations) {
      //       vm.$store.commit('setDonationTotal', resp.data)
      //       vm.loadingDonationMetrics = true
      //     }
      //   }
      // }, err => {
      //   console.log('call error')
      //   console.log(err.status + ': ' + err.statusText)
      // })
    },
    setLang (lang) {
      console.log('LANGUAGE ' + lang)
      this.$i18n.set(lang)
      localStorage.setItem('user_locale', lang)
      this.langDirection = this.getLangDir()
      this.helpLink = this.setHelpUrl(lang)
      this.lang = lang
    },
    getLang () {
      return localStorage.getItem('user_locale')
    },
    // getWalletBalance () {},
    goSharePage (e) {
      e.preventDefault()
      // this.$store.commit('setCurrentPage', 'share')
      this.$events.emit('goToPageEvent', 'share')
    },
    goToPrevPage (e) {
      e.preventDefault()
      // this.$store.commit('goToPreviousPage')
      if (!(this.$store.getters.getCurrentState === 'login' || this.$store.getters.getCurrentState === '')) {
        // this.$store.commit('setCurrentPage', 'home')
        this.$events.emit('goToPageEvent', 'home')
        return
      }
      // this.$store.commit('setCurrentPage', '')
      this.$events.emit('goToPageEvent', '')
    },
    goToNextPage (e) {
      e.preventDefault()
      this.$store.commit('goToNextPage')
    },
    getLangDir () {
      var lang = localStorage.getItem('user_locale')
      console.log('lang ' + lang)
      console.log('dir ' + this.langDirection)
      if (lang !== 'ar') {
        return 'ltr'
      }
      return 'rtl'
    },
    setHelpUrl (lang) {
      var url = 'https://github.com/YoQuieroAyudar/fundraising-API-web-app/wiki/'
      if (lang === 'ar') {
        return url + 'مساعدة'
      } else if (lang === 'fr') {
        return url + 'Aide'
      } else if (lang === 'es') {
        return url + 'Ayuda'
      } else {
        return url + 'Help'
      }
    }
  },
  computed: {
    currentLangDirection () {
      return this.getLangDir()
    },
    currentState () {
      return this.$store.getters.getCurrentState
    },
    getMyBalance () {
      // var vm = this
      // setTimeout(() => {vm.getWalletBalance()}, 20000)
      return this.$store.getters.getBalance
    },
    getWidthClass () {
      var width = (window.innerWidth > 0) ? window.innerWidth : screen.width
      if (width > 479) {
        return 'desktop-max'
      }
      return 'mobile-full-page'
    },
    loggedin () {
      var token = localStorage.getItem('user_token')
      if (token) {
        this.$store.commit('setToken', token)
        this.current_state = 'loggedin'
      }
      return this.$store.getters.getLogin
    },
    isLoggedIn () {
      return this.$store.getters.getLogin
    }
  },
  components: {
    'home-page': Home,
    'share-page': Share,
    'login-form': Login,
    'signup-form': Signup,
    'signup-pos-form': SignupPOS,
    'logout-button': Logout,
    'message-items': Messages,
    'associations-page': Associations,
    'association-page': Association,
    'donations-page': Donations,
    'solidarity-account-page': SolidarityAccount,
    'settings-page': Settings,
    'slide-page': Slides
  }
}

</script>

<style scoped>
#wrapper {
  position: relative;
  margin: auto;
  border: 1px solid #555;
  height: 500px;
  padding: 0.5em;
  overflow: hidden;
}
.mobile-full-page {
  max-width: 100%;
}
.desktop-max {
  max-width: 300px;
}
h1, h2, h3, h4, h5 {
  font-weight: bold;
}
h1 {
  font-size: 1.6em;
}
h2{
  font-size: 1.45em;
}
h3{
  font-size: 1.3em;
}
h4 {
  font-size: 1.15em;
}
h5 {
  font-size: 1em;
}
.content {
  height: 90%;
  overflow-y: scroll;
  overflow-x: hidden;
  padding-right: .4em;
}
a {
  cursor: pointer;
}
.top-container {
  position: relative;
  box-shadow: 1px 2px 2px #CCC;
  padding: 0;
  margin: .2em;
}
.top-menu {
  position: relative;
}
.btn-plain {
  border-top-right-radius: 0;
  border-bottom-right-radius: 0;
  color: #000;
  background-color: #fff;
}
.bottom-menu {
  position: absolute;
  height: 1.5em;
  padding: .3em;
  margin: .85em;
  bottom: 0;
  left: 0;
  max-width: 290px;
  width: 92%;
}
.bottom-menu .test-mode {
  color: #FFF;
  background-color: darkgreen;
}
.bottom-menu .version {
  text-align: center;
  margin: 0;
  padding: .1em;
  width: 100%;
}
#title {
  text-align: center;
  margin: auto;
}
.page-title {
  border: 1px solid #888;
}
.langs {
  z-index: 999;
  margin-top: -1em;
  clear: both;
}
.btn-back {
  margin: .3em;
  padding: .3em;
  color: #000;
  background: #AAA;
}
</style>
