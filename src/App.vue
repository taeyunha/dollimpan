<template>
  <v-app>
    <v-row align="center">
      <v-col>
        <dollimpan
            v-model="items"
            :speed="parseInt(speed)"
            :rolling="parseInt(rolling)"
            :pan-size="panSize"
            :font-size="fontSize"
            @done="handleDone"
        ></dollimpan>
      </v-col>
      <v-col cols="auto" class="controls">
        <h1>행운의 돌림판</h1>
        <h6>(돌림판을 마우스로 돌려보세요)</h6>
        <v-tabs v-model="activeTab" fixed-tabs>
          <v-tab>설정</v-tab>
          <v-tab>히스토리</v-tab>
        </v-tabs>
        <v-tabs-items class="settings" v-model="activeTab">
          <v-tab-item>
            <v-row no-gutters>
              <v-text-field hide-details label="시간(초)" v-model="speed" type="number" min="1"></v-text-field>
              <v-text-field hide-details label="회전수" v-model="rolling" type="number" min="0"></v-text-field>
            </v-row>
            <v-slider v-model="panSize" label="판 크기" max="85"></v-slider>
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
          </v-tab-item>

          <v-tab-item>
            <v-row no-gutters><v-col><v-switch v-model="exceptWhenPicked" label="당첨시 돌림판에서 제외"></v-switch></v-col></v-row>
            <v-row no-gutters>
              <v-list width="100%">
                <v-list-item
                    v-for="(historyItem,i) in [...historyItems].reverse()" :key="i"
                    two-line
                >
                  <v-list-item-content>
                    <v-list-item-title>{{historyItem.value}}</v-list-item-title>
                    <v-list-item-subtitle>{{historyItem.date}}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-row>
          </v-tab-item>
        </v-tabs-items>
      </v-col>
    </v-row>
  </v-app>
</template>

<style lang="scss">
html{
  overflow: hidden;
}
  #app{
    overflow:hidden;
  }
  .settings {
    min-width: 400px;
    min-height: 500px;
    max-height: 600px;
    overflow-y:auto;
    overflow-x:hidden;
  }
  .controls{
    padding-right:100px !important;
  }
</style>

<script>
import Dollimpan from "@/components/Dollimpan";
export default {
  name: 'App',

  components: {
    Dollimpan
  },

  data: () => ({
    exceptWhenPicked: false,
    activeTab: 0,

    historyItems: [],
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
    handleDone(result) {
      result.date = this.$day().format('YYYY-MM-DD HH:mm:ss')
      this.historyItems.push(result)
      if(this.exceptWhenPicked){
        this.items = this.items.filter(item => item.value !== result.value)
      }
    },
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
