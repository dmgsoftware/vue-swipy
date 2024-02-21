<template>
  <div
    ref="interactElement"
    :style="{
      transform: transformString,
      transition: transitionString,
      touchAction: 'none',
    }"
  >
    <slot />
  </div>
</template>

<script>
import interact from "interactjs";

const INTERACT_ON_START = "start";
const INTERACT_ON_MOVE = "move";
const INTERACT_ON_END = "end";

const SWIPE_NONE = "swipe-none";
const SWIPE_LEFT = "swipe-left";
const SWIPE_RIGHT = "swipe-right";
const SWIPE_ANY = "swipe";

export default {
  name: "Swipeable",
  props: {
    transition: {
      type: String,
      default: "transform 0.5s cubic-bezier(0.2, 0.8, 0.4, 1.2)",
      required: false,
    },
    maxRotation: {
      type: Number,
      default: 15,
      required: false,
    },
    outOfSightXOffset: {
      type: Number,
      default: 500,
      required: false,
    },
    thresholdX: {
      type: Number,
      default: 50,
      required: false,
    },
    initialStatus: {
      type: String,
      default: SWIPE_NONE,
      required: false,
    },
  },
  data() {
    return {
      isDragging: false,
      interactPosition: {
        x: 0,
        y: 0,
        rotation: 0,
      },
      status: {
        type: String,
        default: SWIPE_NONE,
        required: false,
      },
    };
  },
  computed: {
    transformString() {
      const { x, y, rotation } = this.interactPosition;
      return `translate3D(${x}px, ${y}px, 0) rotate(${rotation}deg)`;
    },
    transitionString() {
      return !this.isDragging && this.$props.transition;
    },
  },
  created() {
    this.status = this.initialStatus;
    const { outOfSightXOffset, maxRotation } = this.$props;
    if (this.status === SWIPE_RIGHT) {
      this.setPosition({
        x: outOfSightXOffset,
        rotation: maxRotation,
      });
    } else if (this.status === SWIPE_LEFT) {
      this.setPosition({
        x: -outOfSightXOffset,
        rotation: -maxRotation,
      });
    } else if (this.status === SWIPE_NONE) {
      this.setPosition({ x: 0, y: 0, rotation: 0 });
    }
  },
  mounted() {
    if (this.status === SWIPE_NONE) {
      this.setInteractElement();
    }
  },
  beforeUpdate() {
    if (this.status !== SWIPE_NONE) {
      this.unsetInteractElement();
    }
  },
  beforeDestroy() {
    this.unsetInteractElement();
  },
  methods: {
    onThresholdReached(interaction) {
      this.unsetInteractElement();

      switch (interaction) {
        case SWIPE_RIGHT:
        case SWIPE_LEFT:
          this.setStatus(interaction);
          this.$emit(interaction);
          break;
      }
      this.$emit(SWIPE_ANY, interaction);
    },
    setPosition(position) {
      const { x = 0, y = 0, rotation = 0 } = position;
      this.interactPosition = { x, y, rotation };
    },
    setInteractElement() {
      const element = this.$refs.interactElement;

      interact(element).draggable({
        onstart: () => {
          this.$emit(INTERACT_ON_START);
          this.isDragging = true;
        },
        onmove: (event) => {
          this.$emit(INTERACT_ON_MOVE);
          const { maxRotation, thresholdX } = this.$props;
          const x = this.interactPosition.x + event.dx;
          const y = this.interactPosition.y + event.dy;

          let rotation = maxRotation * (x / thresholdX);
          if (rotation > maxRotation) rotation = maxRotation;
          else if (rotation < -maxRotation) rotation = -maxRotation;

          this.setPosition({ x, y, rotation });
        },
        onend: () => {
          this.$emit(INTERACT_ON_END);
          const { x } = this.interactPosition;
          const { thresholdX } = this.$props;
          this.isDragging = false;

          if (x > thresholdX) this.onThresholdReached(SWIPE_RIGHT);
          else if (x < -thresholdX) this.onThresholdReached(SWIPE_LEFT);
          else this.setPosition({ x: 0, y: 0, rotation: 0 });
        },
      });
    },
    unsetInteractElement() {
      interact(this.$refs.interactElement).unset();
    },
    setStatus(status) {
      if (this.status === status) {
        return;
      }

      this.status = status;
      const { outOfSightXOffset, maxRotation } = this.$props;

      if (this.status === SWIPE_RIGHT) {
        this.setPosition({
          x: outOfSightXOffset,
          rotation: maxRotation,
        });
      } else if (this.status === SWIPE_LEFT) {
        this.setPosition({
          x: -outOfSightXOffset,
          rotation: -maxRotation,
        });
      } else if (this.status === SWIPE_NONE) {
        this.setPosition({ x: 0, y: 0, rotation: 0 });
        this.setInteractElement();
      }
    },
    getStatus() {
      return this.status;
    },
  },
};
</script>
