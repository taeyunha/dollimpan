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
          :style="`transform: rotate(${item.degree + unit / 2}deg);`"
      >{{item.value}}
        <div class="line"
             :style="`transform: rotate(${unit / 2}deg);`"
        ></div>
      </div>
    </div>
    <div class="btn">
      <v-btn
          @click="handleClickRoll"
      >돌려!</v-btn>
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
      items: this.value,
      unit : 0,
      panDegree: 0,
      speed: 1, //초
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
      this.panDegree += Math.random() * 360 + 360 * 2 // 0 ~ 0.99999 === 0 ~ 359.9999

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
      window.setTimeout(
          this.showResult,
          this.speed * 1000,
          result
      )
    },
    spreadItem(){
      const items = this.items
      const unit = 360 / items.length
      this.unit = unit
      items.forEach((item,index) => {
        item.degree = unit * index
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

  .item{
    position:absolute;
    transform-origin: bottom;
    height:50%;
    display:flex;
    justify-content: center;
    .line{
      transform-origin: bottom;
      position: absolute;
      top:0;
      height:100%;
      border-right: 1px solid black;
    }
  }

  .pan{
    border: 5px solid black;
    height: 50vh;
    width: 50vh;
    border-radius: 25vh;
    position:relative;

    box-shadow: 0 0 10px grey;
    display:flex;
    justify-content: center;

    transition: transform ease-out; // speed 필요함
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