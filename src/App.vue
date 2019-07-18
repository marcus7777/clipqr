<template>
  <div id="app">
    <qrcode-stream @init="onInit" @decode="onDecode"></qrcode-stream>
    <div v-for="item in list">
      {{item}} <button v-clipboard="item"> Copy </button>
    </div>
  </div>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'
import Clipboard from 'v-clipboard'

export default {
  name: "app",
  components: {
    QrcodeStream,
    Clipboard,
  },
  data: () => ({
    list:[],
  }),
  methods: {
    onDecode(decodedString) {
      console.log(decodedString);
      this.list.push(decodedString)
    },
    async onInit(promise) {
    // show loading indicator

      try {
        await promise;

        // successfully initialized
      } catch (error) {
        if (error.name === "NotAllowedError") {
          // user denied camera access permisson
        } else if (error.name === "NotFoundError") {
          // no suitable camera device installed
        } else if (error.name === "NotSupportedError") {
          // page is not served over HTTPS (or localhost)
        } else if (error.name === "NotReadableError") {
          // maybe camera is already in use
        } else if (error.name === "OverconstrainedError") {
          // did you requested the front camera although there is none?
        } else if (error.name === "StreamApiNotSupportedError") {
          // browser seems to be lacking features
        }
      } finally {
        // hide loading indicator
      }
    }
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
