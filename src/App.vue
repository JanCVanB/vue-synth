<template lang="pug">
  div#app
    synth
</template>

<script>
import Synth from './components/Synth'

export default {

  name: 'app',

  components: {
    Synth
  }

}
</script>

<style lang="sass">
html, body, #app
  height: 100%
  margin: 0
  padding: 0
  width: 100%
</style>
