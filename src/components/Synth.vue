<template lang="pug">
  div.synth
    el-radio-group.instruments(v-model='selectedInstrument')
      el-radio-button(
        v-for='instrument in instruments'
        v-bind:key='instrument'
        v-bind:label='instrument'
      )
    .keys
      el-button.key(
        v-for='note in notes'
        v-bind:class='{ "is-down": note.isPlaying, "is-flat": note.isFlat }'
        v-bind:key='note.frequency'
        v-on:mousedown.native='play(note)'
        v-on:mouseup.native='stop(note)'
      )
        span.name
          | {{ note.name }}
          span.flat(v-show='note.isFlat') &#9837;
          sub {{ note.octave }}
</template>

<script>
const SINE = 'Sine'
const PIANO = 'Piano'
const VIOLIN = 'Violin'
const INSTRUMENTS = [SINE, PIANO, VIOLIN]

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
      instruments: INSTRUMENTS,
      notes: notesData.map(noteDatum => ({
        ...noteDatum,
        isPlaying: false,
        waaHarmonicOscillators: [],
        waaHarmonicGains: [],
        waaTotalGain: null
      })),
      selectedInstrument: INSTRUMENTS[0],
      waaAudioContext: new window.AudioContext()
    }
  },

  computed: {

    harmonicGainValues () {
      switch (this.selectedInstrument) {
        case SINE:
          return [1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
        case PIANO:
          return [1, 0.339, 0.229, 0.152, 0.197, 0.094, 0.061, 0.139, 0.011, 0.071]
        case VIOLIN:
          return [1, 0.287, 0.150, 0.043, 0.204, 0.229, 0.157, 0.115, 0.000, 0.097]
      }
    }

  },

  watch: {

    harmonicGainValues (newHarmonicGainValues) {
      this.notes.forEach(note => {
        note.waaHarmonicGains.forEach((harmonicGain, harmonicIndex) => {
          harmonicGain.gain.value = newHarmonicGainValues[harmonicIndex]
        })
        note.waaTotalGain.gain.value = note.isPlaying ? this.getNoteTotalGainValue() : 0
      })
    }

  },

  methods: {

    getMagnitude (vector) {
      return Math.sqrt(vector.reduce(
        (accumulator, each) => accumulator + Math.pow(each, 2)
      ))
    },

    getNoteTotalGainValue () {
      return 0.5 / this.getMagnitude(this.harmonicGainValues)
    },

    play (note) {
      note.isPlaying = true
      note.waaTotalGain.gain.value = this.getNoteTotalGainValue()
    },

    setUpWaaNodes () {
      this.notes.forEach(note => {
        const totalGain = this.waaAudioContext.createGain()
        for (let harmonic = 1; harmonic <= 10; harmonic++) {
          const harmonicOscillator = this.waaAudioContext.createOscillator()
          harmonicOscillator.frequency.value = note.frequency * harmonic
          harmonicOscillator.start(0)
          const harmonicGain = this.waaAudioContext.createGain()
          harmonicGain.gain.value = this.harmonicGainValues[harmonic - 1]
          harmonicOscillator.connect(harmonicGain)
          harmonicGain.connect(totalGain)
          note.waaHarmonicOscillators.push(harmonicOscillator)
          note.waaHarmonicGains.push(harmonicGain)
        }
        totalGain.gain.value = 0
        totalGain.connect(this.waaAudioContext.destination)
        note.waaTotalGain = totalGain
      })
    },

    stop (note) {
      note.isPlaying = false
      note.waaTotalGain.gain.value = 0
    }

  },

  mounted () {
    this.setUpWaaNodes()
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

.instruments
  display: block
  font-family: system-ui
  margin-bottom: 20px

.key
  border-color: rgb(191, 203, 217)
  border-width: 1px 1px 2px 1px
  color: rgb(31, 45, 61)
  font-size: 7pt
  height: 300px
  margin: 10px 0
  padding: 5px
  position: relative
  vertical-align: top
  width: 30px
  &.is-flat
    background-color: #333
    border-color: #000
    color: #fff
    height: 200px
  &.is-down
    border-color: rgb(29, 144, 230)
    color: rgb(29, 144, 230)
  .name
    bottom: 50px
    left: 50%
    position: absolute
    transform: translate(-50%, 50%)

.flat
  font-size: 6pt
</style>
