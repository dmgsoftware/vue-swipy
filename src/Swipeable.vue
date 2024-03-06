<script lang="ts">
import '@interactjs/auto-start'
import '@interactjs/actions/drag'
import interact from '@interactjs/interact'
import type { PropType } from 'vue'

enum INTERACT {
  START = 'start',
  MOVE = 'move',
  END = 'end'
}

export enum SWIPE {
  NONE = 'swipe-none',
  LEFT = 'swipe-left',
  RIGHT = 'swipe-right',
  ANY = 'swipe'
}

export default {
  name: 'Swipeable',
  props: {
    transition: {
      type: String,
      default: 'transform 0.5s cubic-bezier(0.2, 0.8, 0.4, 1.2)',
      required: false
    },
    maxRotation: {
      type: Number,
      default: 15,
      required: false
    },
    outOfSightXOffset: {
      type: Number,
      default: 500,
      required: false
    },
    thresholdX: {
      type: Number,
      default: 50,
      required: false
    },
    initialStatus: {
      type: String as PropType<SWIPE>,
      default: SWIPE.NONE,
      required: false
    }
  },
  data() {
    return {
      isDragging: false,
      interactPosition: {
        x: 0,
        y: 0,
        rotation: 0
      },
      status: this.initialStatus
    }
  },
  computed: {
    transformString() {
      const { x, y, rotation } = this.interactPosition
      return `translate3D(${x}px, ${y}px, 0) rotate(${rotation}deg)`
    },
    transitionString() {
      return !this.isDragging ? this.$props.transition : 'none'
    }
  },
  created() {
    const { outOfSightXOffset, maxRotation } = this.$props
    if (this.status === SWIPE.RIGHT) {
      this.setPosition({
        x: outOfSightXOffset,
        y: 0,
        rotation: maxRotation
      })
    } else if (this.status === SWIPE.LEFT) {
      this.setPosition({
        x: -outOfSightXOffset,
        y: 0,
        rotation: -maxRotation
      })
    } else if (this.status === SWIPE.NONE) {
      this.setPosition({ x: 0, y: 0, rotation: 0 })
    }
  },
  mounted() {
    if (this.status === SWIPE.NONE) {
      this.setInteractElement()
    }
  },
  beforeUpdate() {
    if (this.status !== SWIPE.NONE) {
      this.unsetInteractElement()
    }
  },
  beforeUnmount() {
    this.unsetInteractElement()
  },
  methods: {
    onThresholdReached(interaction: SWIPE) {
      this.unsetInteractElement()

      switch (interaction) {
        case SWIPE.RIGHT:
        case SWIPE.LEFT:
          this.setStatus(interaction)
          this.$emit(interaction)
          break
      }
      this.$emit(SWIPE.ANY, interaction)
    },
    setPosition(position: typeof this.interactPosition = { x: 0, y: 0, rotation: 0 }) {
      this.interactPosition = position
    },
    setInteractElement() {
      const element = this.$refs.interactElement as HTMLDivElement

      interact(element).draggable({
        onstart: () => {
          this.$emit(INTERACT.START)
          this.isDragging = true
        },
        onmove: (event) => {
          this.$emit(INTERACT.MOVE)
          const { maxRotation, thresholdX } = this.$props
          const x = this.interactPosition.x + event.dx
          const y = this.interactPosition.y + event.dy

          let rotation = maxRotation * (x / thresholdX)
          if (rotation > maxRotation) rotation = maxRotation
          else if (rotation < -maxRotation) rotation = -maxRotation

          this.setPosition({ x, y, rotation })
        },
        onend: () => {
          this.$emit(INTERACT.END)
          const { x } = this.interactPosition
          const { thresholdX } = this.$props
          this.isDragging = false

          if (x > thresholdX) this.onThresholdReached(SWIPE.RIGHT)
          else if (x < -thresholdX) this.onThresholdReached(SWIPE.LEFT)
          else this.setPosition()
        }
      })
    },
    unsetInteractElement() {
      interact(this.$refs.interactElement as HTMLDivElement).unset()
    },
    setStatus(status: SWIPE) {
      if (this.status === status) {
        return
      }

      this.status = status
      const { outOfSightXOffset, maxRotation } = this.$props

      if (this.status === SWIPE.RIGHT) {
        this.setPosition({
          x: outOfSightXOffset,
          y: 0,
          rotation: maxRotation
        })
      } else if (this.status === SWIPE.LEFT) {
        this.setPosition({
          x: -outOfSightXOffset,
          y: 0,
          rotation: -maxRotation
        })
      } else if (this.status === SWIPE.NONE) {
        this.setPosition()
        this.setInteractElement()
      }
    },
    getStatus() {
      return this.status
    }
  }
}
</script>

<template>
  <div
    ref="interactElement"
    :style="{
      transform: transformString,
      transition: transitionString,
      touchAction: 'none'
    }"
  >
    <slot />
  </div>
</template>
