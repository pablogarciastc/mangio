<template>
  <v-card class="mx-auto">
    <v-img :src="img" height="150px"></v-img>

    <v-row>
      <v-col md="3" style="margin: auto; padding-left:30px;">
        <img height="150" :src="cabocha" />
      </v-col>
      <v-col md="9" style="margin: auto ">
        <v-card-subtitle style="font-size: 120%">
          Tu índice de masa corporal es <b>{{this.$store.getters.bmi}}</b>: según la OMS eres una persona que <b>{{this.$store.getters.bmiText}}</b>.
          Además, tu BMR (energía que gasta tu cuerpo en sus funciones vitales diariamente) es de <b>{{this.$store.getters.bmr}}</b> calorías.
          Se incrementa en <b>{{this.$store.getters.tdeeBmr}}</b> calorías por <b>{{this.$store.getters.tdeeText}}</b> física que realizas. 
          <br><br>
          Haz click para saber más.
        </v-card-subtitle>
      </v-col>
    </v-row>

    <v-card-actions id="card-action">
      <v-spacer></v-spacer>

      <v-btn icon @click="show = !show">
        <v-icon>{{ show ? "mdi-chevron-up" : "mdi-chevron-down" }}</v-icon>
      </v-btn>
    </v-card-actions>

    <v-expand-transition>
      <div v-show="show">
        <v-divider></v-divider>
        <v-card-text style="font-size: 100%">
          {{this.$store.getters.macroText}}
          <br><br>
          {{this.$store.getters.exerciseRec}}
        </v-card-text>
      </div>
    </v-expand-transition>
  </v-card>
</template>

<script>
  export default {
    data () {
      return {
       name: 'explanation-card',
       img: require('../../assets/hq_fruit.png'),
       cabocha: require('../../assets/colaboza.png'),
       show: false,
      };
    },
    methods: {
       getPercentages(sums){
         var values_sums = []
         values_sums.push(sums["proteina"])
         values_sums.push(sums["grasa"])
         values_sums.push(sums["carbohidratos"])

          var pctg = []
          var total = values_sums.reduce((a, b) => parseInt(a) + parseInt(b), 0)
          values_sums.forEach( e => {
            pctg.push( (e*100 / total).toFixed(0))
          })
          return pctg
        }
    },
  }
</script>

<style scoped>
  .v-card {
    margin-top: 2rem;
  }

  #card-action {
    margin-top: -3rem;
  }
</style>
