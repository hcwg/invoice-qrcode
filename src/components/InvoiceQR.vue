<template>
  <div class="hello">
    <h1>Invoice QRCode reader</h1>
    <video id="video" width="100%" height="400px"></video>
    <form class="pure-form pure-form-aligned">
      <fieldset>
        <div class="pure-control-group">
          <select id="camera-select" @change="onCameraSelect" v-model="selectedDeviceId">
            <option
              v-for="camera in videoInputDevices"
              v-bind:key="camera.deviceId"
              v-bind:value="camera.deviceId"
            >{{camera.label}}</option>
          </select>
        </div>
      </fieldset>
    </form>
    <button class="button-error pure-button" @click="onClear">Clear All({{resultCount}})</button>
    <button class="pure-button" @click="onCopy">Copy</button>

    <transition-group name="fade" tag="ul">
      <li v-for="item in scanResults" v-bind:key="item">{{item}}</li>
    </transition-group>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import { BrowserQRCodeReader, NotFoundException, ChecksumException } from '@zxing/library'
import 'purecss'
import * as ls from 'local-storage'
import copy from 'copy-to-clipboard'

const storageKey = 'scanResults'
@Component

export default class InvoiceQR extends Vue {
  @Prop() private msg!: string;
  codeReader: BrowserQRCodeReader = new BrowserQRCodeReader()
  videoInputDevices: MediaDeviceInfo[] = []
  selectedDeviceId: string = ''
  scanResults: string[] = []

  mounted () {
    this.scanResults = ls.get<string[]>(storageKey) || []

    this.codeReader
      .listVideoInputDevices()
      .then(videoInputDevices => {
        this.videoInputDevices = videoInputDevices
        videoInputDevices.forEach(device =>
          console.log(`${device.label}, ${device.deviceId}`)
        )
        if (this.videoInputDevices.length) {
          const lastCamera = ls.get<string>('selectedCamera')
          const selectedDevice = this.videoInputDevices.find((value, index, arr) => value.deviceId === lastCamera) || this.videoInputDevices[0]
          this.selectedDeviceId = selectedDevice.deviceId
          this.startScan()
        }
      })
      .catch(err => console.error(err))
  }
  startScan () {
    if (this.selectedDeviceId === '') {
      return
    }
    this.codeReader
      .decodeFromInputVideoDeviceContinuously(this.selectedDeviceId, 'video', (result, err) => {
        if (err) {
          if (err instanceof NotFoundException || err instanceof ChecksumException) {
            return
          }
          console.error(err)
        } else {
          const text = result.getText()
          if (this.scanResults.indexOf(text) === -1) {
            this.scanResults.unshift(result.getText())
            ls.set<string[]>(storageKey, this.scanResults)
          }
        }
      })
  }

  onClear () {
    const clear = confirm('Are you sure to clear all scan result')
    if (clear) {
      this.scanResults = []
      ls.set<string[]>(storageKey, this.scanResults)
    }
  }
  onCameraSelect () {
    ls.set<string>('selectedCamera', this.selectedDeviceId)
    this.startScan()
  }

  onCopy () {
    copy(this.scanResults.join('\n'))
  }

  get resultCount () {
    return this.scanResults.length
  }
}
</script>

<style scoped>
.button-success,
.button-error,
.button-warning,
.button-secondary {
  color: white;
  border-radius: 4px;
  text-shadow: 0 1px 1px rgba(0, 0, 0, 0.2);
}

.button-success {
  background: rgb(28, 184, 65); /* this is a green */
}

.button-error {
  background: rgb(202, 60, 60); /* this is a maroon */
}

.button-warning {
  background: rgb(223, 117, 20); /* this is an orange */
}

.button-secondary {
  background: rgb(66, 184, 221); /* this is a light blue */
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s;
  background-color: antiquewhite;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
li {
  word-break: break-all;
}
</style>
