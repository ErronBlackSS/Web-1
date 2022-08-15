<template>
  <div
    class="px-5"
  >
    <button @click.stop="gotoReglamentContent">
      Пройти регламент "{{ name }}"
    </button>
  </div>
</template>

<script>
export default {
  props: {
    name: {
      type: String,
      default: null
    },
    uid: {
      type: String,
      default: null
    }
  },
  computed: {
    storeNavigator () {
      return this.$store.state.navigator.navigator
    }
  },
  methods: {
    gotoReglamentContent () {
      this.$router.push('/directory')
      // asidemenu logic
      this.$store.commit('updateStackWithInitValue', {
        name: 'Регламенты',
        key: 'greedSource',
        greedPath: 'reglaments',
        value: this.storeNavigator.reglaments?.items
      })
      this.$store.commit('basic', { key: 'greedSource', value: this.storeNavigator.reglaments?.items })
      this.$store.commit('basic', { key: 'mainSectionState', value: 'greed' })
      this.$store.commit('basic', { key: 'greedPath', value: 'reglaments' })
      const reglament = this.$store.state.reglaments.reglaments[this.uid]
      this.$store.commit('basic', {
        key: 'reglamentSource',
        value: { uid: '92413f6c-2ef3-476e-9429-e76d7818685d', param: this.uid }
      })
      const navElem = {
        name: this.name,
        key: 'greedSource',
        uid: this.uid,
        global_property_uid: '92413f6c-2ef3-476e-9429-e76d7818685d',
        greedPath: 'reglament_content',
        value: []
      }

      this.$store.commit('pushIntoNavStack', navElem)
      this.$store.commit('basic', { key: 'greedSource', value: reglament })
      this.$store.commit('basic', { key: 'greedPath', value: 'reglament_content' })
    }
  }
}
</script>
