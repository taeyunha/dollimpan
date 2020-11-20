<template>
  <v-app>
    <v-row align="center">
      <v-col>
        <dollimpan
            v-model="items"
            :speed="speed"
            :rolling="rolling"
            :pan-size="panSize"
            :font-size="fontSize"
        ></dollimpan>
      </v-col>
      <v-col cols="3">
        <v-row>
          <v-text-field hide-details label="시간(초)" v-model="speed" type="number" min="1"></v-text-field>
          <v-text-field hide-details label="회전수" v-model="rolling" type="number" min="0"></v-text-field>
        </v-row>
        <v-slider v-model="panSize" label="판 크기"></v-slider>
        <v-slider v-model="fontSize" label="글자 크기" max="10"></v-slider>
        <v-row no-gutters align="center">
          <v-col><v-text-field v-model="range[0]" type="number" @change="handleRangeChange"></v-text-field></v-col>
          ~
          <v-col><v-text-field v-model="range[1]" type="number" @change="handleRangeChange"></v-text-field></v-col>
        </v-row>
        <v-textarea
            v-model="input"
            solo
            @change="input = input.trim()"
        ></v-textarea>
        <v-btn @click="shuffle">셔플</v-btn>
        <v-btn @click="generate">생성</v-btn>
      </v-col>
    </v-row>
  </v-app>
</template>

<script>
import Dollimpan from "@/components/Dollimpan";
export default {
  name: 'App',

  components: {
    Dollimpan
  },

  data: () => ({
    items: [],
    input: '0\n1\n2\n3\n4\n5\n6\n7\n8\n9',
    range: [0,9],
    speed: 5, //초
    rolling: 2,
    panSize: 50,
    fontSize: 3.5
  }),
  mounted () {
    this.generate()
  },
  methods: {
    shuffle() {
      const input = this.input.split('\n')
      const shuffled = []
      for(; input.length;){
        console.debug(input.length)
        shuffled.push(...input.splice(parseInt(Math.random()*input.length),1))
      }
      this.input = shuffled.join('\n')
    },
    handleRangeChange(){
      const newInput = []
      for(let i = this.range[0]; i <= this.range[1]; i++){
        newInput.push(i)
      }
      this.input = newInput.join('\n')
    },
    generate(){
      this.items = this.input.split('\n').map(value => ({value}))
      // {value: }
    }
  }
};
</script>
