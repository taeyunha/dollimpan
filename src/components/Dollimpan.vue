<template>
  <div class="dollimpan">
    <v-btn
        class="btn-roll"
        @click="handleClickRoll"
    >돌려!
    </v-btn>
    <div v-if="currentItem" class="result" :style="`font-size: ${Math.max(fontSize,3.5)}vh; top: -${Math.max(fontSize,3.5)}vh`">{{currentItem.value}}</div>
    <div class="arrow"></div>
    <div class="pan"
         ref="pan"
         :style="`transform: rotate(${panDegree}deg); transition-duration: ${grab ? 0 : speed}s; height: ${panSize}vh; width: ${panSize}vh;`"
         @mousedown="handleMouseDown"
         @mousemove="handleMouseMove"
         @mouseup="handleLeave"
         @mouseleave="handleLeave"
    >
      <div
          v-for="(item) in items" :key="item.key"
          class="item"
          :style="`transform: rotate(${item.degree + unit / 2}deg)`"
      >
        <div class="background"
             :style="`border-top-color: ${item.color || 'none'}; border-width: ${backgroundWidth}vh; border-top-width: ${panSize/2}vh;`"
        ></div>
        <span class="value" :style="`font-size:${fontSize}vh;`">{{ item.value }}</span>
        <div class="line"
             :style="`transform: rotate(${unit / 2}deg);`"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Dollimpan",
  props: {
    value: Array,
    panSize: Number,
    fontSize: Number,
    speed: Number,
    rolling: Number
  },
  data () {
    return {
      toRadian: (degree) => degree * Math.PI / 180,
      toDegree: (radian) => radian / (Math.PI / 180),
      items: this.value,
      currentItem: null,
      unit: 0,
      panDegree: 0,

      timeout: -1,
      grab: false,
      grabStart: null,

      backgroundWidth: 1,
      interval: null
    }
  },
  watch: {
    value (val) {
      this.items = val
      this.spreadItem();
    },
    panSize () {
      this.spreadItem()
    }
  },
  beforeDestroy () {
    window.clearInterval(this.interval)
  },
  mounted () {
    this.spreadItem();
  },
  methods: {
    getRotationAngle (target) {
      const obj = window.getComputedStyle(target, null);
      const matrix = obj.getPropertyValue('-webkit-transform') ||
          obj.getPropertyValue('-moz-transform') ||
          obj.getPropertyValue('-ms-transform') ||
          obj.getPropertyValue('-o-transform') ||
          obj.getPropertyValue('transform');

      let angle = 0;

      if (matrix !== 'none') {
        const values = matrix.match(/[-\d.]+/g);
        const cos = values[0];
        const sin = values[1];
        angle = Math.acos(cos) * 180 / Math.PI;
        sin < 0 && (angle = 360 - angle);
      }

      return angle;
    },
    calculateDelta (e) {
      const {left, top, width, height} = this.$refs.pan.getBoundingClientRect()
      const O = {x: left + width / 2, y: top + height / 2}
      const {clientX: sx, clientY: sy} = this.grabStart
      const {clientX: ex, clientY: ey} = e
      const originVector = this.makeVector({x: sx - O.x, y: sy - O.y})
      const newVector = this.makeVector({x: ex - O.x, y: ey - O.y})
      return (originVector.theta - newVector.theta) * (O.y > ey ? 1 : -1)
    },
    makeVector ({x, y}) {
      const scalar = Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2))
      const theta = this.toDegree(Math.acos(x / scalar))
      return {
        x, y, scalar, theta
      }
    },
    handleMouseMove (e) {
      if (this.grab) {
        this.panDegree += this.calculateDelta(e)

        const {clientX: ex, clientY: ey} = e
        this.grabStart = {...this.grabStart, clientX: ex, clientY: ey, time: new Date().getTime()}
        //console.debug(distance)
        this.$forceUpdate()
      }
    },
    handleLeave (e) {
      if (this.grab && this.grabStart) {
        const {time: st} = this.grabStart

        const time = new Date().getTime() - st
        const delta = this.calculateDelta(e)

        this.grab = false
        this.grabStart = null
        const velocity = delta / time
        if (velocity !== 0) {
          const result = this.roll(velocity * this.speed * 1000)
          this.showResult(result)
        }
        console.debug('leave', time, delta, velocity)
      }

    },
    handleMouseDown (e) {
      const {clientX, clientY} = e
      this.grab = true
      this.grabStart = {clientX, clientY, time: new Date().getTime()}
      window.clearTimeout(this.timeout)

    },
    showResult (result) {
      window.clearTimeout(this.timeout)
      this.timeout = window.setTimeout(
          () => {
            this.$alert(result.value)
            window.clearInterval(this.interval)
            this.currentItem = result
          },
          this.speed * 1000
      )
    },
    calculateResult (degree) {
      let resDeg = ((degree || this.panDegree) % 360)
      resDeg < 0 && (resDeg += 360)
      resDeg = 360 - resDeg
      return this.items.find((item) => {
        const start = item.degree
        const end = item.degree + this.unit
        return resDeg >= start && resDeg < end
      })
    },
    roll (addDegree) {
      addDegree = addDegree || Math.random() * 360 + 360 * this.rolling
      console.debug('roll', addDegree)
      this.panDegree += addDegree
      window.clearInterval(this.interval)
      this.interval = window.setInterval(() => {
        const degree = this.getRotationAngle(this.$refs.pan)
        this.currentItem = this.calculateResult(degree)
      },5)
      return this.calculateResult()
    },
    handleClickRoll () {
      console.debug('roll click')
      const result = this.roll()
      this.showResult(result)
    },
    spreadItem () {
      const items = this.items
      const unit = 360 / items.length
      this.unit = unit

      const tan90m$d2 = Math.tan(this.toRadian(90 - unit / 2))// tan(90-@/2)
      const size = (this.panSize / 2)
      this.backgroundWidth = size / tan90m$d2
      const colorUnit = 255 * 5 / (items.length - 1)
      items.forEach((item, index) => {
        item.degree = unit * index
        const x = colorUnit * index
        const R = Math.min(255, Math.max(Math.abs(x - 255 * 3) - 255 * 1, 0))
        const G = Math.min(255, Math.max(-Math.abs(x - 255 * 2) + 255 * 2, 0))
        const B = Math.min(255, Math.max(x - 255 * 2, 0))
        item.color = `rgba(${R},${G},${B},0.8)`

        item.key = Math.floor(Math.random() * 100) + `${item.degree}`
      })
      this.$forceUpdate()
    }
  }
}
</script>

<style scoped lang="scss">
.dollimpan {
  display: flex;
  align-items: center;
  flex-direction: column;
  position: relative;

  .result{
    position: absolute;
    margin-top: -10px;
  }

  .btn-roll {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    z-index: 1;
  }

  .item {
    position: absolute;
    transform-origin: bottom;
    height: 50%;
    display: flex;
    justify-content: center;

    .value {
      color: white;
      text-shadow: 0 0 5px black;
      margin-top: 5px;
    }

    .background {
      position: absolute;
      z-index: -1;
      border-style: solid;
      border-bottom-width: 0 !important;
      border-color: transparent;
    }

    .line {
      transform-origin: bottom;
      position: absolute;
      top: 0;
      height: 100%;
      border-right: 1px solid black;
    }
  }

  .pan {
    user-select: none;
    border-radius: 100vh;
    position: relative;
    overflow: hidden;

    box-shadow: 0 0 10px 5px black;
    display: flex;
    justify-content: center;

    transition: transform cubic-bezier(.18, .55, .35, 1); // speed 필요함

    cursor: move;
    cursor: grab;

    &:active {
      cursor: grabbing;
    }
  }

  .arrow {
    display: flex;
    justify-content: center;
    height: 15px;

    &:after {
      content: "";
      position: absolute;
      border-top: 15px solid black;
      border-left: 5px solid transparent;
      border-right: 5px solid transparent;
      border-bottom: 0;
    }
  }
}
</style>