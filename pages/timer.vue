<template>
  <section>
    <h1 class="title">{{ $store.state.timer }} seconds</h1>
    <div class="field">
      <div class="control">
        <masked-input
          class="input is-large"
          type="text"
          placeholder="0m00s"
          v-model="$store.state.time"
          :guide="true"
          :mask="[/\d/, 'm', /\d/, /\d/, 's']"
          placeholderChar="_">
        </masked-input>
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
      <a id="safari" class="hidden" v-on:click="speak"></a>
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
  import MaskedInput from 'vue-text-mask'

  const AudioConstructor = {}
  let isSafari = false

  Vue.use(Vuex)

  export default {
    components: {
      MaskedInput
    },
    store: new Vuex.Store({
      state: {
        timer: 0,
        time: '',
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
          state.time = ''
          state.timeInterval = null
          state.bufferSource = null
        }
      },
      actions: {
        startCount ({commit, state, dispatch}) {
          let timeRegex = /(\d)m(\d+)s/g
          let [, minutes, seconds] = timeRegex.exec(state.time)
          state.timer = (parseInt(minutes) * 60) + parseInt(seconds)
          state.timeInterval = setInterval(function () {
            commit('countdown')
            if (state.timer === 0) {
              clearInterval(state.timeInterval)
              dispatch('startAlarm')
            }
            if ((state.timer % 10 === 0 || state.timer <= 5) && state.timer > 0) {
              if (isSafari) {
                let event = new MouseEvent('click', {
                  'view': window,
                  'bubbles': true,
                  'cancelable': true
                })
                let btn = document.getElementById('safari')
                btn.dispatchEvent(event)
              } else {
                dispatch('speak')
              }
            }
          }, 1000)
        },
        startAlarm ({state}) {
          if (state.audioBuffer) {
            let context = new AudioConstructor.AudioContext()
            let bufferSource = context.createBufferSource()
            let gainNode = context.createGain()
            gainNode.gain.value = 50
            bufferSource.buffer = state.audioBuffer
            bufferSource.connect(gainNode)
            gainNode.connect(context.destination)
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
        resetState ({commit, dispatch}) {
          dispatch('stopAlarm')
          commit('reset')
        },
        speak ({state}) {
          const synth = AudioConstructor.SpeechSynthesis
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
            let utterThis = new AudioConstructor.SpeechSynthesisUtterance(state.timer)
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
      },
      speak: function () {
        this.$store.dispatch('speak')
      }
    },
    mounted () {
      isSafari = /Safari/.test(navigator.userAgent) && /Apple Computer/.test(navigator.vendor)
      AudioConstructor.AudioContext = 'webkitAudioContext' in window ? window.webkitAudioContext : window.AudioContext
      AudioConstructor.SpeechSynthesis = window.speechSynthesis || window.SpeechSynthesis || window.webkitspeechSynthesis || window.webkitSpeechSynthesis
      AudioConstructor.SpeechSynthesisUtterance = window.SpeechSynthesisUtterance || window.webkitSpeechSynthesisUtterance
      let context = new AudioConstructor.AudioContext()
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
