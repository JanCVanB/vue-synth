<template lang="pug">
  div.synth
    el-button.key(
      v-for='note in notes'
      v-bind:class='{ "is-down": note.isPlaying }'
      v-bind:key='note.frequency'
      v-on:mousedown.native='play(note)'
      v-on:mouseup.native='stop(note)'
    )
      | {{ note.name }}
      sub {{ note.octave }}
</template>

<script>
export default {

  name: 'Synth',

  data () {
    return {
      notes: [
        { name: 'A', octave: 3, frequency: 220.000, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'B', octave: 3, frequency: 246.942, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'C', octave: 4, frequency: 261.626, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'D', octave: 4, frequency: 293.665, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'E', octave: 4, frequency: 329.628, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'F', octave: 4, frequency: 349.228, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'G', octave: 4, frequency: 391.995, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'A', octave: 4, frequency: 440.000, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'B', octave: 4, frequency: 493.883, isPlaying: false, waaOscillator: null, waaGain: null },
        { name: 'C', octave: 5, frequency: 523.251, isPlaying: false, waaOscillator: null, waaGain: null }
      ],
      waaAudioContext: new window.AudioContext()
    }
  },

  methods: {

    createwaaOscillators () {
      this.notes.forEach(note => {
        note.waaOscillator = this.waaAudioContext.createOscillator()
        note.waaOscillator.frequency.value = note.frequency
        note.waaOscillator.start(0)
        note.waaGain = this.waaAudioContext.createGain()
        note.waaGain.gain.value = 0
        note.waaOscillator.connect(note.waaGain)
        note.waaGain.connect(this.waaAudioContext.destination)
      })
    },

    play (note) {
      note.isPlaying = true
      note.waaGain.gain.value = 1
    },

    stop (note) {
      note.isPlaying = false
      note.waaGain.gain.value = 0
    }

  },

  mounted () {
    this.createwaaOscillators()
  }

}
</script>

<style lang="sass" scoped>
.synth
  box-sizing: border-box
  height: 100%
  padding: 100px
  text-align: center
  width: 100%

.key
  border-color: rgb(191, 203, 217)
  color: rgb(31, 45, 61)
  &.is-down
    border-color: rgb(29, 144, 230)
    color: rgb(29, 144, 230)
</style>
