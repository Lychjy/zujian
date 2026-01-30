<template>
  <div class="comparison" :style="rootStyle">
    <div
      ref="frameRef"
      class="frame"
      aria-label="Image comparison slider"
      @pointerdown="onPointerDown"
      @pointermove="onPointerMove"
      @pointerup="onPointerUp"
      @pointercancel="onPointerUp"
    >
      <div class="layer before" :style="{ backgroundImage: 'url(' + beforeSrc + ')' }"></div>
      <div class="layer after" :style="{ backgroundImage: 'url(' + afterSrc + ')' }"></div>

      <div v-if="beforeLabel" class="label before">{{ beforeLabel }}</div>
      <div v-if="afterLabel" class="label after">{{ afterLabel }}</div>

      <div class="divider" aria-hidden="true">
        <div class="handle" aria-hidden="true">
          <svg viewBox="0 0 24 24" aria-hidden="true">
            <path d="M7.41 7.41 6 8.83 9.17 12 6 15.17l1.41 1.42L12 12z"></path>
            <path d="M16.59 7.41 12 12l4.59 4.59L18 15.17 14.83 12 18 8.83z"></path>
          </svg>
        </div>
      </div>
    </div>

    <div class="range">
      <input
        :value="String(pos)"
        type="range"
        min="0"
        max="100"
        aria-label="Comparison position"
        @input="onRangeInput"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'ImageCompareSliderVue2',
  props: {
    beforeSrc: {
      type: String,
      required: true,
    },
    afterSrc: {
      type: String,
      required: true,
    },
    beforeLabel: {
      type: String,
      default: 'Before',
    },
    afterLabel: {
      type: String,
      default: 'After',
    },
    value: {
      type: Number,
      default: 50,
    },
    aspectRatio: {
      type: String,
      default: '16 / 9',
    },
  },
  data() {
    return {
      dragging: false,
      pos: this.clamp(Number(this.value) || 0, 0, 100),
    }
  },
  computed: {
    rootStyle() {
      return {
        '--slider-pos': this.pos + '%',
        '--aspect-ratio': this.aspectRatio,
      }
    },
  },
  watch: {
    value(v) {
      const next = this.clamp(Number(v) || 0, 0, 100)
      if (next !== this.pos) this.pos = next
    },
  },
  beforeDestroy() {
    this.dragging = false
  },
  methods: {
    clamp(v, min, max) {
      return Math.min(max, Math.max(min, v))
    },
    setPos(p, shouldEmit = true) {
      const next = this.clamp(p, 0, 100)
      if (next === this.pos) return
      this.pos = next
      if (shouldEmit) {
        this.$emit('input', next)
        this.$emit('change', next)
      }
    },
    posFromClientX(clientX) {
      const el = this.$refs.frameRef
      if (!el) return this.pos
      const rect = el.getBoundingClientRect()
      const x = this.clamp(clientX - rect.left, 0, rect.width)
      return (x / rect.width) * 100
    },
    onPointerDown(e) {
      const el = this.$refs.frameRef
      if (!el) return
      this.dragging = true
      try {
        el.setPointerCapture(e.pointerId)
      } catch (err) {
        // ignore
      }
      this.setPos(this.posFromClientX(e.clientX))
    },
    onPointerMove(e) {
      if (!this.dragging) return
      this.setPos(this.posFromClientX(e.clientX))
    },
    onPointerUp() {
      this.dragging = false
    },
    onRangeInput(e) {
      const v = Number(e && e.target && e.target.value)
      this.setPos(isFinite(v) ? v : 0)
    },
  },
}
</script>

<style scoped>
.comparison {
  width: min(1000px, 100%);
}

.frame {
  position: relative;
  width: 100%;
  aspect-ratio: var(--aspect-ratio);
  border-radius: 16px;
  overflow: hidden;
  background: #111827;
  box-shadow: 0 16px 50px rgba(0, 0, 0, 0.55);
  user-select: none;
  touch-action: none;
}

.layer {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}

.layer.after {
  clip-path: inset(0 calc(100% - var(--slider-pos)) 0 0);
}

.divider {
  position: absolute;
  top: 0;
  bottom: 0;
  left: var(--slider-pos);
  width: 0;
  transform: translateX(-1px);
  pointer-events: none;
}

.divider::before {
  content: "";
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  width: 2px;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 0 0 1px rgba(0, 0, 0, 0.35);
}

.handle {
  position: absolute;
  top: 50%;
  left: 0;
  transform: translate(-50%, -50%);
  width: 44px;
  height: 44px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.55);
  display: grid;
  place-items: center;
}

.handle svg {
  width: 22px;
  height: 22px;
  fill: #0b0f17;
  opacity: 0.95;
}

.label {
  position: absolute;
  top: 14px;
  padding: 8px 10px;
  border-radius: 999px;
  font-size: 12px;
  line-height: 1;
  letter-spacing: 0.2px;
  background: rgba(0, 0, 0, 0.45);
  border: 1px solid rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(8px);
  color: #e7eaf0;
}

.label.before {
  left: 14px;
}

.label.after {
  right: 14px;
}

.range {
  margin-top: 14px;
  width: 100%;
}

input[type='range'] {
  width: 100%;
  accent-color: #e7eaf0;
}
</style>
