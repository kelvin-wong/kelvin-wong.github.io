<template>
  <section>
    <h1 class="title">{{ $store.state.timer }} seconds</h1>
    <div class="field">
      <div class="control">
        <input class="input is-large" type="number" v-model.number="$store.state.time" step="0.1" placeholder="minutes">
      </div>
    </div>
    <div class="field is-grouped is-grouped-centered">
      <div class="control">
        <a class="button is-large is-primary" v-on:click="countdown">Go</a>
      </div>
      <div class="control">
        <a class="button is-large is-warning" v-on:click="stop">Stop</a>
      </div>
      <div class="control">
        <a class="button is-large is-danger" v-on:click="reset">Reset</a>
      </div>
    </div>
  </section>
</template>

<style>
  .button {
    text-transform: uppercase;
  }
</style>

<script>
  import Vue from 'vue'
  import Vuex from 'vuex'

  Vue.use(Vuex)

  export default {
    store: new Vuex.Store({
      state: {
        timer: 0,
        time: null,
        timeInterval: null,
        audioBuffer: null,
        bufferSource: null
      },
      mutations: {
        countdown (state) {
          state.timer--
        },
        reset (state) {
          clearInterval(state.timeInterval)
          state.timer = 0
          state.time = null
          state.timeInterval = null
          state.bufferSource = null
        }
      },
      actions: {
        startCount ({commit, state, dispatch}) {
          state.timer = parseInt(state.time * 60)
          state.timeInterval = setInterval(function () {
            commit('countdown')
            if (state.timer === 0) {
              clearInterval(state.timeInterval)
              dispatch('startAlarm')
            }
            if ((state.timer % 10 === 0 || state.timer <= 5) && state.timer > 0) {
              dispatch('speak')
            }
          }, 1000)
        },
        startAlarm ({state}) {
          if (state.audioBuffer) {
            let context = new AudioContext()
            let bufferSource = context.createBufferSource()
            bufferSource.buffer = state.audioBuffer
            bufferSource.connect(context.destination)
            bufferSource.start(0)
            state.bufferSource = bufferSource
          }
        },
        stopAlarm ({state}) {
          if (state.bufferSource) {
            state.bufferSource.stop(0)
          }
        },
        resetState ({commit}) {
          commit('reset')
        },
        speak ({state}) {
          const synth = window.speechSynthesis
          const voices = synth.getVoices()
          let voice
          voices.forEach(function (v) {
            if (v.lang.toLowerCase() === 'zh-hk') {
              voice = v
            }
            if (voice === null && v.lang.toLowerCase() === 'en-us') {
              voice = v
            }
          })
          setTimeout(function () {
            let utterThis = new SpeechSynthesisUtterance(state.timer)
            utterThis.voice = voice
            utterThis.pitch = 1.2
            utterThis.rate = 0.8
            utterThis.volume = 1
            synth.speak(utterThis)
          }, 0)
        }
      }
    }),
    head: {
      title: 'Workout Timer'
    },
    methods: {
      countdown: function () {
        this.$store.dispatch('startCount')
      },
      stop: function () {
        this.$store.dispatch('stopAlarm')
      },
      reset: function () {
        this.$store.dispatch('resetState')
      }
    },
    mounted () {
      let context = new AudioContext()
      let request = new XMLHttpRequest()
      let that = this
      request.open('GET', '/alarm.mp3', true)
      request.responseType = 'arraybuffer'
      request.onload = function () {
        context.decodeAudioData(request.response, function (buffer) {
          that.$store.state.audioBuffer = buffer
          context.close()
        }, function (err) {
          console.dir(err)
        })
      }
      request.send()
    }
  }
</script>