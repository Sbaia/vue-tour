<template>
  <div class="v-tour">
    <slot
      :current-step="currentStep"
      :steps="steps"
      :previous-step="previousStep"
      :next-step="nextStep"
      :stop="stop"
      :isFirst="isFirst"
      :isLast="isLast"
    >
      <!--Default slot {{ currentStep }}-->
      <v-step
        v-if="currentStep === index"
        v-for="(step, index) of steps"
        :key="index"
        :step="step"
        :previous-step="previousStep"
        :next-step="nextStep"
        :stop="stop"
        :isFirst="isFirst"
        :isLast="isLast"
        :skipLabel="skipLabel"
        :previousLabel="previousLabel"
        :nextLabel="nextLabel"
        :finishLabel="finishLabel"

      >
        <span slot="content" slot-scope="{ step }">
          <slot name="content" :step="step">
            <div v-if="step.content" v-html="step.content"></div>
            <div v-else>This is a demo step! The id of this step is {{ hash }} and it targets {{ step.target }}.</div>
          </slot>
        </span>
        <span slot="header" slot-scope="{ step }">
          <slot name="header" :step="step">
            <div v-if="step.header" class="v-step__header">
              <div v-if="step.header.title" v-html="step.header.title"></div>
            </div>
          </slot>
        </span>
        <span slot="actions" slot-scope="{ step, stop, previousStep, nextStep, isLast, isFirst }">
          <slot name="actions" :step="step" :stop="stop" :previousStep="previousStep" :nextStep="nextStep" :isLast="isLast" :isFirst="isFirst">
            <div class="v-step__buttons">
              <button @click="stop" v-if="!isLast" class="v-step__button">
                {{ skipLabel }}
              </button>
              <button @click="previousStep" v-if="!isFirst" class="v-step__button">
                {{ previousLabel }}
              </button>
              <button @click="nextStep" v-if="!isLast" class="v-step__button">
                {{ nextLabel }}
              </button>
              <button @click="stop" v-if="isLast" class="v-step__button">
                {{ finishLabel }}
              </button>
            </div>
          </slot>
        </span>
        <!--<div v-if="index === 2" slot="actions">
          <a @click="nextStep">Next step</a>
        </div>-->
      </v-step>
    </slot>
  </div>
</template>

<script>
import { DEFAULT_CALLBACKS, DEFAULT_OPTIONS, KEYS } from '../shared/constants'

export default {
  name: 'v-tour',
  props: {
    steps: {
      type: Array,
      default: () => []
    },
    name: {
      type: String
    },
    options: {
      type: Object,
      default: () => { return DEFAULT_OPTIONS }
    },
    callbacks: {
      type: Object,
      default: () => { return DEFAULT_CALLBACKS }
    }
  },
  data () {
    return {
      currentStep: -1
    }
  },
  mounted () {
    this.$tours[this.name] = this

    if (this.customOptions.useKeyboardNavigation) {
      window.addEventListener('keyup', this.handleKeyup)
    }
  },
  beforeDestroy () {
    // Remove the keyup listener if it has been defined
    if (this.customOptions.useKeyboardNavigation) {
      window.removeEventListener('keyup', this.handleKeyup)
    }
  },
  computed: {
    // Allow us to define custom options and merge them with the default options.
    // Since options is a computed property, it is reactive and can be updated during runtime.
    customOptions () {
      return {
        ...DEFAULT_OPTIONS,
        ...this.options
      }
    },
    customCallbacks () {
      return {
        ...DEFAULT_CALLBACKS,
        ...this.callbacks
      }
    },
    // Return true if the tour is active, which means that there's a VStep displayed
    isRunning () {
      return this.currentStep > -1 && this.currentStep < this.numberOfSteps
    },
    isFirst () {
      return this.currentStep === 0
    },
    isLast () {
      return this.currentStep === this.steps.length - 1
    },
    numberOfSteps () {
      return this.steps.length
    },
    skipLabel () {
      return this.customOptions.skipLabel
    },
    previousLabel () {
      return this.customOptions.previousLabel
    },
    nextLabel () {
      return this.customOptions.nextLabel
    },
    finishLabel () {
      return this.customOptions.finishLabel
    }
  },
  methods: {
    start () {
      // Wait for the DOM to be loaded, then start the tour
      setTimeout(() => {
        this.customCallbacks.onStart()
        this.currentStep = 0
      }, this.customOptions.startTimeout)
    },
    previousStep () {
      if (this.currentStep > 0) {
        this.customCallbacks.onPreviousStep(this.currentStep)
        this.currentStep--
      }
    },
    nextStep () {
      if (this.currentStep < this.numberOfSteps - 1 && this.currentStep !== -1) {
        this.customCallbacks.onNextStep(this.currentStep)
        this.currentStep++
      }
    },
    stop () {
      this.customCallbacks.onStop()
      this.currentStep = -1
    },

    handleKeyup (e) {
      // TODO: debug mode
      // console.log('[Vue Tour] A keyup event occured:', e)
      switch (e.keyCode) {
        case KEYS.ARROW_RIGHT:
          this.nextStep()
          break
        case KEYS.ARROW_LEFT:
          this.previousStep()
          break
        case KEYS.ESCAPE:
          this.stop()
          break
      }
    }
  }
}
</script>
