<template>
  <div id="app">
    <wbfc-linkage :url="action">
    </wbfc-linkage>
  </div>
</template>

<script>
import WbfcLinkage from './WbfcLinkage.vue'

export default {
  name: 'App',
  components: {
    WbfcLinkage
  },
  data () {
    return {
      action: 'http://192.168.20.5:1106/system/area/linkageList'
    }
  }
}
</script>
