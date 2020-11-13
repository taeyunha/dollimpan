<template>
  <div class="dollimpan">
    <h1>행운의 돌림판</h1>
    <div class="arrow"></div>
    <div class="pan"
         :style="`transform: rotate(${panDegree}deg); transition-duration: ${speed}s`"
    >
      <div
          v-for="(item,index) in items" :key="index"
          class="item"
          :style="`transform: rotate(${item.degree + unit / 2}deg)`"
      >
        <div class="background"
             :style="`border-top-color: ${item.color || 'none'}; border-width: ${item.width}vh;`"
        ></div>
        <span class="value">{{item.value}}</span>
        <div class="line"
             :style="`transform: rotate(${unit / 2}deg);`"
        ></div>
      </div>
    </div>
    <div class="btn">
      <v-row justify="center">
        <v-btn
            @click="handleClickRoll"
        >돌려!</v-btn>
      </v-row>
      <v-row>
        <v-text-field hide-details label="시간(초)" v-model="speed" type="number" min="1"></v-text-field>
        <v-text-field hide-details label="회전수" v-model="rolling" type="number" min="0"></v-text-field>
      </v-row>
    </div>
  </div>
</template>

<script>
export default {
  name: "Dollimpan",
  props: {
    value: Array
  },
  data() {
    return {
      radian: (degree) => degree * Math.PI / 180,
      items: this.value,
      unit : 0,
      panDegree: 0,
      speed: 5, //초
      rolling: 2,
      timeout: -1
    }
  },
  watch: {
    value (val) {
      this.items = val
      this.spreadItem();
    }
  },
  mounted () {
    this.spreadItem();
  },
  methods: {
    showResult(result){
      this.$alert(result.value)
    },
    roll() {
      this.panDegree += Math.random() * 360 + 360 * this.rolling // 0 ~ 0.99999 === 0 ~ 359.9999

      const resDeg = 360 - (this.panDegree % 360)
      return this.items.find((item) => {
        const start = item.degree
        const end = item.degree + this.unit
        return resDeg >= start && resDeg < end
      })
    },
    handleClickRoll() {
      console.debug('roll click')
      const result = this.roll()
      window.clearTimeout(this.timeout)
      this.timeout = window.setTimeout(
          this.showResult,
          this.speed * 1000,
          result
      )
    },
    spreadItem(){
      const items = this.items
      const unit = 360 / items.length
      this.unit = unit

      const tan90m$d2 = Math.tan(this.radian(90 - unit/2))// tan(90-@/2)
      const colorUnit = 255 * 5 / (items.length-1)
      items.forEach((item,index) => {
        item.degree = unit * index
        const x = colorUnit * index
        const R = Math.min(255,Math.max(Math.abs(x - 255 * 3) - 255 * 1 ,0))
        const G = Math.min(255,Math.max(-Math.abs( x - 255 * 2) + 255 * 2, 0))
        const B = Math.min(255,Math.max(x - 255*2, 0))
        item.color = `rgba(${R},${G},${B},0.8)`


        item.width = 25/tan90m$d2
      })
      this.$forceUpdate()
    }
  }
}
</script>

<style scoped lang="scss">
.dollimpan{
  display:flex;
  align-items: center;
  flex-direction: column;
  position: relative;

  .btn{ margin-top: 20px}

  .item{
    position:absolute;
    transform-origin: bottom;
    height:50%;
    display:flex;
    justify-content: center;
    align-items: center;

    .value{
      font-size:3.5vh;
      color: white;
      text-shadow: 0 0 5px black;
      margin-top: -3.5vh;
    }
    .background {
      position:absolute;
      z-index: -1;
      border-style: solid;
      border-top-width: 25vh !important;
      border-bottom-width: 0 !important;
      border-color: transparent;
    }
    .line{
      transform-origin: bottom;
      position: absolute;
      top:0;
      height:100%;
      border-right: 1px solid black;
    }
  }

  .pan{
    height: 50vh;
    width: 50vh;
    border-radius: 25vh;
    position:relative;
    overflow:hidden;

    box-shadow: 0 0 10px 5px black;
    display:flex;
    justify-content: center;

    transition: transform cubic-bezier(.18,.55,.35,1); // speed 필요함
  }

  .arrow{
    display:flex;
    justify-content: center;
    height: 15px;
    &:after{
      content: "";
      position:absolute;
      border-top: 15px solid black;
      border-left: 5px solid transparent;
      border-right: 5px solid transparent;
      border-bottom: 0;
    }
  }
}
</style>