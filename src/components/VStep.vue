<template>
  <div class="v-step" :id="'v-step-' + hash" :ref="'v-step-' + hash">
    <slot name="header" :step="step">
    </slot>

    <slot name="content" :step="step">
    </slot>

    <slot name="actions" :step="step" :stop="stop" :previousStep="previousStep" :nextStep="nextStep" :isLast="isLast" :isFirst="isFirst">
    </slot>

    <div class="v-step__arrow" x-arrow :class="{ 'v-step__arrow--dark': step.header && step.header.title }"></div>
  </div>
</template>

<script>
import Popper from 'popper.js'
import sum from 'hash-sum'
import { DEFAULT_STEP_OPTIONS } from '../shared/constants'

export default {
  name: 'v-step',
  props: {
    step: {
      type: Object
    },
    previousStep: {
      type: Function
    },
    nextStep: {
      type: Function
    },
    stop: {
      type: Function
    },
    isFirst: {
      type: Boolean
    },
    isLast: {
      type: Boolean
    }
  },
  data () {
    return {
      hash: sum(this.step.target)
    }
  },
  computed: {
    params () {
      return {
        ...DEFAULT_STEP_OPTIONS,
        ...this.step.params
      }
    }
  },
  mounted () {
    let targetElement = document.querySelector(this.step.target)

    // TODO: debug mode
    // console.log('[Vue Tour] The target element ' + this.step.target + ' of .v-step[id="' + this.hash + '"] is:', targetElement)

    if (targetElement) {
      targetElement.scrollIntoView({behavior: 'smooth'})

      /* eslint-disable no-new */
      new Popper(
        targetElement,
        this.$refs['v-step-' + this.hash],
        this.params
      )
    } else {
      console.warn('[Vue Tour] The target element ' + this.step.target + ' of .v-step[id="' + this.hash + '"] does not exist! Skip to next step')
      // this.stop()
      this.nextStep();
    }
  }
}
</script>

<style lang="scss" scoped>
  .v-step {
    background: #50596c; /* #ffc107, #35495e */
    color: white;
    max-width: 320px;
    border-radius: 3px;
    filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.5));
    padding: 1rem;
    text-align: center;
    position: absolute;
    left: -1000;
    top: -1000;
  }

  .v-step .v-step__arrow {
    width: 0;
    height: 0;
    border-style: solid;
    position: absolute;
    margin: 0.5rem;
  }

  .v-step .v-step__arrow {
    border-color: #50596c; /* #ffc107, #35495e */

    &--dark {
      border-color: #454d5d;
    }
  }

  .v-step[x-placement^="top"] {
    margin-bottom: 0.5rem;
  }

  .v-step[x-placement^="top"] .v-step__arrow {
    border-width: 0.5rem 0.5rem 0 0.5rem;
    border-left-color: transparent;
    border-right-color: transparent;
    border-bottom-color: transparent;
    bottom: -0.5rem;
    left: calc(50% - 1rem);
    margin-top: 0;
    margin-bottom: 0;
  }

  .v-step[x-placement^="bottom"] {
    margin-top: 0.5rem;
  }

  .v-step[x-placement^="bottom"] .v-step__arrow {
    border-width: 0 0.5rem 0.5rem 0.5rem;
    border-left-color: transparent;
    border-right-color: transparent;
    border-top-color: transparent;
    top: -0.5rem;
    left: calc(50% - 1rem);
    margin-top: 0;
    margin-bottom: 0;
  }

  .v-step[x-placement^="right"] {
    margin-left: 0.5rem;
  }

  .v-step[x-placement^="right"] .v-step__arrow {
    border-width: 0.5rem 0.5rem 0.5rem 0;
    border-left-color: transparent;
    border-top-color: transparent;
    border-bottom-color: transparent;
    left: -0.5rem;
    top: calc(50% - 1rem);
    margin-left: 0;
    margin-right: 0;
  }

  .v-step[x-placement^="left"] {
    margin-right: 0.5rem;
  }

  .v-step[x-placement^="left"] .v-step__arrow {
    border-width: 0.5rem 0 0.5rem 0.5rem;
    border-top-color: transparent;
    border-right-color: transparent;
    border-bottom-color: transparent;
    right: -0.5rem;
    top: calc(50% - 1rem);
    margin-left: 0;
    margin-right: 0;
  }

  /* Custom */

  .v-step__header {
    margin: -1rem -1rem 0.5rem;
    padding: 0.5rem;
    background-color: #454d5d;
    border-top-left-radius: 3px;
    border-top-right-radius: 3px;
  }

  .v-step__content {
    margin: 0 0 1rem 0;
  }

  .v-step__button {
    background: transparent;
    border: .05rem solid white;
    border-radius: .1rem;
    color: white;
    cursor: pointer;
    display: inline-block;
    font-size: .8rem;
    height: 1.8rem;
    line-height: 1rem;
    outline: none;
    padding: .35rem .4rem;
    text-align: center;
    text-decoration: none;
    transition: all .2s ease;
    vertical-align: middle;
    white-space: nowrap;

    &:hover {
      background-color: rgba(white, 0.95);
      color: #50596c;
    }
  }
</style>
