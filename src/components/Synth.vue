<template lang="pug">
  div.synth
    el-button.key(
      v-for='note in notes'
      v-bind:class='{ "is-down": note.isPlaying, "is-flat": note.isFlat }'
      v-bind:key='note.frequency'
      v-on:mousedown.native='play(note)'
      v-on:mouseup.native='stop(note)'
    )
      | {{ note.name }}
      span.flat(v-show='note.isFlat') &#9837;
      sub {{ note.octave }}
</template>

<script>
const c2 = 65.4064
const equalTemperamentRatio = 1.059463
const noteNames = ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B']
let notesData = []
for (let i = 0; i <= 48; i++) {
  notesData.push({
    frequency: c2 * Math.pow(equalTemperamentRatio, i),
    name: noteNames[i % 12][0],
    isFlat: noteNames[i % 12].includes('b'),
    octave: (i - (i % 12)) / 12 + 2
  })
}

export default {

  name: 'Synth',

  data () {
    return {
      notes: notesData.map(noteDatum => ({
        ...noteDatum,
        isPlaying: false,
        waaOscillator: null,
        waaGain: null
      })),
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
  vertical-align: top
  width: 100%

.key
  border-color: rgb(191, 203, 217)
  color: rgb(31, 45, 61)
  font-size: 7pt
  height: 100px
  margin: 10px
  padding: 5px
  width: 30px
  &.is-flat
    background-color: #000
    border-color: #000
    color: #fff
  &.is-down
    border-color: rgb(29, 144, 230)
    color: rgb(29, 144, 230)

.flat
  font-size: 6pt
</style>
