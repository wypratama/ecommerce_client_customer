<template>
<div class="fixed z-10 inset-0 overflow-y-auto ">
  <div class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
    <!--
      Background overlay, show/hide based on modal state.
      Entering: "ease-out duration-300"
        From: "opacity-0"
        To: "opacity-100"
      Leaving: "ease-in duration-200"
        From: "opacity-100"
        To: "opacity-0"
    -->
    <div class="fixed inset-0 transition-opacity" aria-hidden="true">
      <div class="absolute inset-0 bg-gray-500 opacity-75"></div>
    </div>
    <ErrorCard v-if="addCartErr" />
    <!-- This element is to trick the browser into centering the modal contents. -->
    <span class="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true">&#8203;</span>
    <!--
      Modal panel, show/hide based on modal state.

      Entering: "ease-out duration-300"
        From: "opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
        To: "opacity-100 translate-y-0 sm:scale-100"
      Leaving: "ease-in duration-200"
        From: "opacity-100 translate-y-0 sm:scale-100"
        To: "opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
    -->
    <div v-on-clickaway="away" class="inline-block align-bottom bg-white text-left shadow-xl mx-auto transform transition-all sm:my-8 sm:align-middle sm:max-w-sm sm:w-full animate__animated animate__wobble" role="dialog" aria-modal="true" aria-labelledby="modal-headline">

    <!-- <div class="bg-white w-full max-w-sm rounded-lg shadow-md overflow-hidden mx-auto"> -->
        <div class="py-4 px-6">
            <h2 class="text-center font-bold text-gray-700 text-3xl">{{title}}</h2>

            <h3 class="mt-1 text-center font-medium text-gray-600 text-xl">You Need to Login to Access Full Features</h3>

            <p class="mt-1 text-center text-gray-500">Login or create account</p>

            <form v-if="!showReg" class="animate__animated animate__fadeIn" @submit.prevent="login">
                <div class="mt-4 w-full">
                    <input v-model="log.email" class="w-full mt-2 py-2 px-4 bg-white text-gray-700 border border-gray-300 rounded block placeholder-gray-500 focus:border-blue-500 focus:outline-none focus:ring" type="email" placeholder="Email Address" aria-label="Email Address">
                </div>

                <div class="mt-4 w-full">
                    <input v-model="log.password" class="w-full mt-2 py-2 px-4 bg-white text-gray-700 border border-gray-300 rounded block placeholder-gray-500 focus:border-blue-500 focus:outline-none focus:ring" type="password" placeholder="Password" aria-label="Password">
                </div>

                <div class="flex justify-between items-center mt-4">
                    <!-- <a href="#" class="text-gray-600 text-sm hover:text-gray-500">Forget Password?</a> -->

                    <button class="py-2 px-4 bg-gray-700 text-white rounded hover:bg-gray-600 focus:outline-none" type="submit">
                        Login
                    </button>
                </div>
            </form>

            <form v-if="showReg" class="animate__animated animate__fadeIn" @submit.prevent="register">
                <div class="mt-4 w-full">
                    <input v-model="reg.username" class="w-full mt-2 py-2 px-4 bg-white text-gray-700 border border-gray-300 rounded block placeholder-gray-500 focus:border-blue-500 focus:outline-none focus:ring" type="text" placeholder="Username" aria-label="Username">
                </div>
                <div class="mt-4 w-full">
                    <input v-model="reg.email" class="w-full mt-2 py-2 px-4 bg-white text-gray-700 border border-gray-300 rounded block placeholder-gray-500 focus:border-blue-500 focus:outline-none focus:ring" type="email" placeholder="Email Address" aria-label="Email Address">
                </div>

                <div class="mt-4 w-full">
                    <input v-model="reg.password" class="w-full mt-2 py-2 px-4 bg-white text-gray-700 border border-gray-300 rounded block placeholder-gray-500 focus:border-blue-500 focus:outline-none focus:ring" type="password" placeholder="Password" aria-label="Password">
                </div>

                <div class="mt-4 w-full">
                    <input v-model="reg.address" class="w-full mt-2 py-2 px-4 bg-white text-gray-700 border border-gray-300 rounded block placeholder-gray-500 focus:border-blue-500 focus:outline-none focus:ring" type="text" placeholder="Address" aria-label="Address">
                </div>

                <div class="flex justify-between items-center mt-4">
                    <!-- <a href="#" class="text-gray-600 text-sm hover:text-gray-500">Forget Password?</a> -->

                    <button class="py-2 px-4 bg-gray-700 text-white rounded hover:bg-gray-600 focus:outline-none" type="submit">
                        Register
                    </button>
                </div>
            </form>
        </div>

        <div class="flex items-center justify-center py-4 bg-gray-100 text-center">
            <span class="text-gray-600 text-sm">{{spanText}} </span>
            <a href="#" class="text-blue-600 font-bold mx-2 text-sm hover:text-blue-500" @click="showReg = !showReg">{{buttonTitle}}</a>
        </div>
    <!-- </div> -->
    </div>
  </div>
</div>
</template>

<script>
import { mapState } from 'vuex'
import axios from '../config/axios'
import { directive as onClickaway } from 'vue-clickaway'
import ErrorCard from '../components/ErrorCard'

export default {
  name: 'Login',
  directives: {
    onClickaway: onClickaway
  },
  components: {
    ErrorCard
  },
  data () {
    return {
      showReg: false,
      log: {
        email: '',
        password: ''
      },
      reg: {
        username: '',
        email: '',
        password: '',
        address: ''
      }
    }
  },
  methods: {
    login () {
      console.log('sampe login')
      axios({
        method: 'post',
        url: '/login-user',
        data: {
          email: this.log.email,
          password: this.log.password
        }
      })
        .then(res => {
          localStorage.setItem('access_token', res.data.access_token)
          this.$store.dispatch('loadUser')
          this.$store.dispatch('loadWishLists')
          this.$router.push('/')
        })
        .catch(error => {
          this.$store.commit('setAddCartErr', error.response.data.error)
          setTimeout(() => { this.$store.commit('setAddCartErr', null) }, 3000)
          if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            console.log(error.response.data)
            console.log(error.response.status)
            console.log(error.response.headers)
          } else if (error.request) {
            // The request was made but no response was received
            // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
            // http.ClientRequest in node.js
            console.log(error.request)
          } else {
            // Something happened in setting up the request that triggered an Error
            console.log('Error', error.message)
          }
        })
    },
    away () {
      this.$router.push('/')
    },
    register () {
      console.log('register')
      axios({
        method: 'post',
        url: '/register',
        data: {
          username: this.reg.username,
          email: this.reg.email,
          password: this.reg.password,
          address: this.reg.address
        }
      })
        .then(res => {
          localStorage.setItem('access_token', res.data.access_token)
          this.$store.dispatch('loadUser')
          this.$router.push('/')
        })
        .catch(error => {
          this.$store.commit('setAddCartErr', error.response.data.errors[0])
          setTimeout(() => { this.$store.commit('setAddCartErr', null) }, 3000)
          if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            console.log(error.response.data)
            console.log(error.response.status)
            console.log(error.response.headers)
          } else if (error.request) {
            // The request was made but no response was received
            // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
            // http.ClientRequest in node.js
            console.log(error.request)
          } else {
            // Something happened in setting up the request that triggered an Error
            console.log('Error', error.message)
          }
        })
    }
  },
  computed: {
    ...mapState({
      title: 'title',
      addCartErr: 'addCartErr'
    }),
    buttonTitle () {
      return this.showReg === false ? 'register' : 'login'
    },
    spanText () {
      return this.showReg === false ? 'Don' + 't have an account?' : 'Have an account?'
    }
  }
}
</script>

<style>

</style>
