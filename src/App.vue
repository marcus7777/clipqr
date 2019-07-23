<template>
  <div id="app">
    <h3>An all local qrCode reader</h3>
    <qrcode-stream @init="onInit" @decode="onDecode"></qrcode-stream>
    <div v-for="item in list">
      {{item}} 
      <button v-clipboard="item">Copy</button>
      <button @click="showQr(item)">show</button>
      <button @click="dropQr(item)">X</button>
    </div>
    <qrcode-vue v-if="qr" size="300" style="margin: 30px 0" :value="qr"></qrcode-vue>
    <input v-model="qr" placeholder="qr text" />
    <button v-if="qr" @click="addQr">+</button>
    <button v-if="qr" @click="signQr">sign</button>
    <button v-if="signed" @click="unsignQr">unsign</button>
    <div v-if="pubKey">
      <h2> Public Key </h2>
      <qrcode-vue size="150" style="margin: 15px 0" :value="pubKey"></qrcode-vue>
    </div>  
    <p>
      Made with love, here's
      <a href="https://www.youtube.com/watch?v=FNB84flBeFg&list=PL5qDJFHzH_j7YO-JrYN2phKf1ye3H8lCt&index=3&t=0s" >how</a>
    </p>
  </div>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'
import Clipboard from 'v-clipboard'
import QrcodeVue from 'qrcode.vue';
import { brotliCompress } from 'zlib';

Storage = window.localStorage;

let snd = new Audio("data:audio/wav;base64,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qJrx4KOQjahZxWbcZgztj2c49nKmkId44S71j0c8eV9yDK6uPRzx5X18eDvjvQ6yKo9ZSS6l//8elePK/Lf//IInrOF/FvDoADYAGBMGb7FtErm5MXMlmPAJQVgWta7Zx2go+8xJ0UiCb8LHHdftWyLJE0QIAIsI+UbXu67dZMjmgDGCGl1H+vpF4NSDckSIkk7Vd+sxEhBQMRU8j/12UIRhzSaUdQ+rQU5kGeFxm+hb1oh6pWWmv3uvmReDl0UnvtapVaIzo1jZbf/pD6ElLqSX+rUmOQNpJFa/r+sa4e/pBlAABoAAAAA3CUgShLdGIxsY7AUABPRrgCABdDuQ5GC7DqPQCgbbJUAoRSUj+NIEig0YfyWUho1VBBBA//uQZB4ABZx5zfMakeAAAAmwAAAAF5F3P0w9GtAAACfAAAAAwLhMDmAYWMgVEG1U0FIGCBgXBXAtfMH10000EEEEEECUBYln03TTTdNBDZopopYvrTTdNa325mImNg3TTPV9q3pmY0xoO6bv3r00y+IDGid/9aaaZTGMuj9mpu9Mpio1dXrr5HERTZSmqU36A3CumzN/9Robv/Xx4v9ijkSRSNLQhAWumap82WRSBUqXStV/YcS+XVLnSS+WLDroqArFkMEsAS+eWmrUzrO0oEmE40RlMZ5+ODIkAyKAGUwZ3mVKmcamcJnMW26MRPgUw6j+LkhyHGVGYjSUUKNpuJUQoOIAyDvEyG8S5yfK6dhZc0Tx1KI/gviKL6qvvFs1+bWtaz58uUNnryq6kt5RzOCkPWlVqVX2a/EEBUdU1KrXLf40GoiiFXK///qpoiDXrOgqDR38JB0bw7SoL+ZB9o1RCkQjQ2CBYZKd/+VJxZRRZlqSkKiws0WFxUyCwsKiMy7hUVFhIaCrNQsKkTIsLivwKKigsj8XYlwt/WKi2N4d//uQRCSAAjURNIHpMZBGYiaQPSYyAAABLAAAAAAAACWAAAAApUF/Mg+0aohSIRobBAsMlO//Kk4soosy1JSFRYWaLC4qZBYWFRGZdwqKiwkNBVmoWFSJkWFxX4FFRQWR+LsS4W/rFRb/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////VEFHAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAU291bmRib3kuZGUAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAMjAwNGh0dHA6Ly93d3cuc291bmRib3kuZGUAAAAAAAAAACU=");  

  function hex (buffer) {
    var hexCodes = []
    var view = new DataView(buffer)
    for (var i = 0; i < view.byteLength; i += 4) {
    // Using getUint32 reduces the number of iterations needed (we process 4 bytes each time)
      var value = view.getUint32(i)
    // toString(16) will give the hex representation of the number without padding
      var stringValue = value.toString(16)
    // We use concatenation and slice for padding
      var padding = '00000000'
      var paddedValue = (padding + stringValue).slice(-padding.length)
      hexCodes.push(paddedValue)
    }
    // Join all the hex strings into one
    return hexCodes.join('')
  }
  function importKey (KeyAsJson, cb) {
    var key = JSON.parse(KeyAsJson)
    var hard = key.hard ? new RegExp(key.hard, 'g') : ''
    return window.crypto.subtle.importKey('jwk', key, {name: 'ECDSA', namedCurve: 'P-256'}, false, key.key_ops).then(function (key) {
      return cb(key, hard)
    }).then(function (ret) {
      return ret
    }).catch(function (e) {
      console.error(e)
    })
  }
  function exportKey (Key, cb) {
    return window.crypto.subtle.exportKey('jwk', Key).then(function (keydata) {
    // returns the exported key data
      var key = JSON.stringify(keydata)
      cb(key)
      return key
    }).then(function (key) {
      return key
    }).catch(function (err) {
      console.error(err)
    })
  }
  function ab2str(buf) {
    return String.fromCharCode.apply(null, new Uint16Array(buf))
  }
  function str2ab(str) {
    var buf = new ArrayBuffer(str.length*2); // 2 bytes for each char
    var bufView = new Uint16Array(buf);
    for (var i=0, strLen=str.length; i < strLen; i++) {
      bufView[i] = str.charCodeAt(i);
    }
    return buf
  }
  function base64ToArrayBuffer(s) {
    var asciiString = window.atob(s)
    return new Uint8Array([...asciiString].map(char => char.charCodeAt(0)))
  }
  function arrayToBase64String (ab) {
    var dView = new Uint8Array(ab)   // Get a byte view
    var arr = Array.prototype.slice.call(dView) // Create a normal array
    var arr1 = arr.map(function (item) {
      return String.fromCharCode(item)    // Convert
    })
    return window.btoa(arr1.join(''))  // Form a string
  }
  function verify(string, signature, publicKey, cb) {
    let data = str2ab(string)
    importKey(publicKey,function(key){
      window.crypto.subtle.verify({name: "ECDSA",hash: {name: "SHA-256"}}, key, base64ToArrayBuffer(signature), data 
      ).then(function(isvalid){
      //returns a boolean on whether the signature is true or not
        cb(isvalid)
      }).catch(function(err){
        console.error(err)
      })
    })
  }

  // this function takes callbacks private key and public key and return {privateKey: "{...}",publicKey: "{...}"}
  function generate (privateKeyCB, publicKeyCB) {
    var keys = {}
    return window.crypto.subtle.generateKey({ name: 'ECDSA', namedCurve: 'P-256'}, true, ['sign', 'verify']).then(function (key) {
      return exportKey(key.privateKey, privateKeyCB || function(){}).then(function (privateKey) {
        keys.privateKey = privateKey
        return exportKey(key.publicKey, publicKeyCB || function(){}).then(function (publicKey) {
          keys.publicKey = publicKey
        })
      }).then(function () {
        return keys
      })
    }).catch(function (e) {
      console.error(e)
    })
  }
  // sign takes a string to be signed and a private key and a call back which is fired with the argument of the signature
  function sign (string, privateKey, cb) {
    var data = str2ab(string)
    return importKey(privateKey, function (key, regx) {
      if (!regx) {
        return window.crypto.subtle.sign({name: 'ECDSA', hash: {name: 'SHA-256'}}, key, data).then(function (signature) {
          return cb(arrayToBase64String(signature))
        }).catch(function (e) {
          console.error(e)
        })
      } else {
        var doWork = function () {
          var work = window.crypto.subtle.sign({name: 'ECDSA', hash: {name: 'SHA-256'}}, key, data)
          return work.then(function (signature) {
            var sig = arrayToBase64String(signature)
            if (regx.test(sig)) {
              return cb(sig)
            } else {
              return doWork()
            }
          })
        }
        return doWork()
      }
    })
  }
export default {
  name: "app",
  components: {
    QrcodeStream,
    Clipboard,
    QrcodeVue,
  },
  data: () => {
    let list = JSON.parse(Storage.getItem('qrList')) || [];
    let prvKey = JSON.parse(Storage.getItem('prvKey')) || "";
    let pubKey = JSON.parse(Storage.getItem('pubKey')) || "";
    return {
      list,
      qr:"",
      prvKey,pubKey,
      signed: false,
    }
  },
  watch: {
    list: {
      handler: function (val) {
        Storage.setItem("qrList", JSON.stringify(val));
      },
      deep: true
    },
    qr: {
      handler: function (qr) {
        try {
          if (qr.slice(-2) === "==" && qr.length > 88 && atob(qr.slice(-88))) {
            let sig = qr.slice( -88);
            let msg = qr.slice(0, -88);
            verify(msg, sig, this.pubKey, function(b){
              this.signed = b;
            }.bind(this));
          } else {
            this.signed = false;
          }
        } catch (error) {
          this.signed = false;
        }
      }
    }
  },
  methods: {
    unsignQr() {
      this.qr = this.qr.slice(0,-88)
    },
    signQr() {
      let saveSign = function(keys) {
        this.pubKey = keys.publicKey;
        this.prvKey = keys.privateKey;
        sign(this.qr, this.prvKey, addSignature);
      }.bind(this)
      let addSignature = function(sig) {
        console.log("cb",sig);
        this.qr =  this.qr + sig
      }.bind(this)
      if (this.prvKey) {
        sign(this.qr, this.prvKey, addSignature);
      } else {
        generate((key) => {
          Storage.setItem("prvKey", JSON.stringify(key));
        },(key) => {
          Storage.setItem("pubKey", JSON.stringify(key));
        }).then(saveSign)
      }
    },
    addQr(){
      this.list.push(this.qr)
      this.list = [...new Set(this.list)];
    },
    dropQr(item){
      var index = this.list.indexOf(item);
      if (index > -1) {
        this.list.splice(index, 1);
      }
    },
    showQr(item){
      this.qr = item;
    },
    onDecode(decodedString) {
      this.list.push(decodedString)
      this.list = [...new Set(this.list)];
      snd.play();
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
