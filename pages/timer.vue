<template>
  <section>
    <h1 class="title">{{ $store.state.timer }} seconds</h1>
    <div class="field has-addons">
      <div class="control">
        <input class="input" type="number" v-model.number="$store.state.time" step="0.1" placeholder="minutes">
      </div>
      <div class="control">
        <a id="go" class="button" v-on:click="countdown">GO</a>
      </div>
    </div>
  </section>
</template>

<style>
  .field.has-addons {
    justify-content: center;
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
        current: 0,
        timeInterval: null
      },
      mutations: {
        countdown (state) {
          state.timer--
        }
      },
      actions: {
        startCount ({commit, state, dispatch}) {
          state.timer = parseInt(state.time * 60)
          state.timeInterval = setInterval(function () {
            commit('countdown')
            if (state.timer === 0) {
              clearInterval(state.timeInterval)
              dispatch('done')
            }
            if ((state.timer % 10 === 0 || state.timer <= 5) && state.timer > 0) {
              dispatch('speak')
            }
          }, 1000)
        },
        done ({state}) {
          // TODO
          let context = new AudioContext()
          let oscillator = context.createOscillator()
          let volumeNode = context.createGain()
          volumeNode.gain.value = 0.5
          oscillator.type = 'sine'
          oscillator.frequency.v1alue = 1000
          oscillator.connect(volumeNode)
          volumeNode.connect(context.destination)
          for (let i = 0; i < 1000; i += 100) {
            volumeNode.gain.setValueAtTime(0, i)
          }
          for (let i = 100; i < 1000; i += 100) {
            volumeNode.gain.setValueAtTime(1, i)
          }
          oscillator.start(0)
          oscillator.stop(1000)
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
    time: 0,
    head: {
      title: 'Workout Timer'
    },
    methods: {
      countdown: function () {
        this.$store.dispatch('startCount')
      }
    }
  }
</script>