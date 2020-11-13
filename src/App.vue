<template>
  <v-app>
    <v-row align="center">
      <v-col>
        <dollimpan
            v-model="items"
        ></dollimpan>
      </v-col>
      <v-col cols="3">
        <v-textarea
            v-model="input"
            solo
            @input="generate"
        ></v-textarea>
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
    input: '0\n1\n2\n3\n4\n5\n6\n7\n8\n9'
  }),
  mounted () {
    this.generate()
  },
  methods: {
    generate(){
      this.items = this.input.split('\n').map(value => ({value}))
      // {value: }
      console.debug(this.items)
    }
  }
};
</script>
