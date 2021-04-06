<template>
  <v-row justify="center" align="center">
    <div class="pt-10">
      <v-alert v-if="error" class="mt-2" type="error">
        {{ error }}
      </v-alert>
      <p v-if="false" class="decode-result">
        Останній результат: <b>{{ result }}</b>
      </p>
      <div v-show="loading">Завантаження...</div>
      <qrcode-stream
        v-show="!loading"
        :camera="camera"
        @decode="onDecode"
        @init="onInit"
      >
        <div v-if="validationSuccess" class="validation-success">
          This is a URL
        </div>

        <div v-if="validationFailure" class="validation-failure">
          This is NOT a URL!
        </div>

        <div v-if="validationPending" class="validation-pending">
          Валідація...
        </div>
      </qrcode-stream>
    </div>
  </v-row>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      isValid: undefined,
      camera: 'auto',
      result: null,
      error: '',
      loading: false,
    }
  },

  computed: {
    validationPending() {
      return this.isValid === undefined && this.camera === 'off'
      // return true
    },

    validationSuccess() {
      return this.isValid === true
    },

    validationFailure() {
      return this.isValid === false
    },
  },

  methods: {
    async onDecode(content) {
      this.result = content
      this.turnCameraOff()

      // pretend it's taking really long
      await this.timeout(3000)
      this.isValid = content.startsWith('http')

      // some more delay, so users have time to read the message
      await this.timeout(2000)

      this.turnCameraOn()
    },

    async onInit(promise) {
      try {
        this.loading = true
        await promise.then(this.resetValidationState)
      } catch (error) {
        if (error.name === 'NotAllowedError') {
          this.error = 'ERROR: you need to grant camera access permisson'
        } else if (error.name === 'NotFoundError') {
          this.error = 'ERROR: no camera on this device'
        } else if (error.name === 'NotSupportedError') {
          this.error = 'ERROR: secure context required (HTTPS, localhost)'
        } else if (error.name === 'NotReadableError') {
          this.error = 'ERROR: is the camera already in use?'
        } else if (error.name === 'OverconstrainedError') {
          this.error = 'ERROR: installed cameras are not suitable'
        } else if (error.name === 'StreamApiNotSupportedError') {
          this.error = 'ERROR: Stream API is not supported in this browser'
        }
      } finally {
        this.loading = false
      }
    },
    resetValidationState() {
      this.isValid = undefined
    },
    turnCameraOn() {
      this.camera = 'auto'
    },

    turnCameraOff() {
      this.camera = 'off'
    },

    timeout(ms) {
      return new Promise((resolve) => {
        window.setTimeout(resolve, ms)
      })
    },
  },
}
</script>

<style scoped>
.error {
  font-weight: bold;
}
.validation-success,
.validation-failure,
.validation-pending {
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  text-align: center;
  font-weight: bold;
  font-size: 1.4rem;
  padding: 10px;
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}
.validation-success {
  color: green;
}
.validation-failure {
  color: red;
}
</style>
