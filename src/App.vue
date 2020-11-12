<template>
  <v-app>
    <v-row align="center">
      <v-col>
        <dollimpan
            v-model="items"
        ></dollimpan>
      </v-col>
      <v-col>
        <v-textarea
            v-model="input"
            solo></v-textarea>
        <v-btn @click="handleClickGenerate">생성!!</v-btn>
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
    input: ''
  }),
  methods: {
    handleClickGenerate(){
      this.items = this.input.split('\n').map(value => ({value}))
      // {value: }
      console.debug(this.items)
    }
  }
};
</script>
