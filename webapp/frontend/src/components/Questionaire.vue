<template>
  <v-container>
    <div>
      <h1 class="title">Cuestionario de preferencias nutricionales</h1>
    </div>
    <v-stepper v-model="e6" vertical class="elevation-0">
      <v-stepper-step :complete="e6 > 1" step="1" color="var(--main-orange)" editable>
        ¿Quién eres?
        <small style="margin-top: 3px"
          >Cuéntanos un poco sobre ti, esto nos ayudará a personalizar tu plan
          de alimentación</small
        >
      </v-stepper-step>

      <v-stepper-content step="1">
        <v-text-field
          v-model="username"
          :counter="30"
          label="Nombre de usuario"
          style="margin-bottom: 10px"
        ></v-text-field>

        <v-text-field
          v-model="name"
          :counter="30"
          label="Nombre"
          style="margin-bottom: 10px"
        ></v-text-field>

        <v-text-field
          v-model="age"
          label="Edad"
          type="number"
          style="margin-bottom: 20px"
        ></v-text-field>

        <v-select
          v-model="selected_sex"
          :items="sex_options"
          label="Selecciona tu sexo"
          solo
        ></v-select>

        <v-slider
          v-model="height.val"
          :label="height.label"
          :thumb-color="height.color"
          color="var(--main-orange)"
          track-color="var(--main-orange-pale)"
          thumb-label="always"
          max="220"
          min="150"
          style="margin-bottom: 20px"
        ></v-slider>
        <v-btn color="var(--main-orange)" @click="e6 = 2"> Continuar </v-btn>
      </v-stepper-content>

      <v-stepper-step :complete="e6 > 2" step="2" color="var(--main-orange)" editable>
        ¿Cuánto pesas?
        <small style="margin-top: 3px">
          Recuerda, puedes conectarte con una balanza Xiaomi para una estimación
          más precisa</small
        >
      </v-stepper-step>

      <v-stepper-content step="2">
        <v-checkbox
          v-model="using_scale"
          label="Va a utilizar la balanza Xiaomi Mi Body Composition Scale 2 ?"
          color="red"
          hide-details
        ></v-checkbox>

        <div v-if="using_scale == true" class="if-balanza">
          <v-btn class="ma-2" 
          outlined color="indigo"
          @click="getBalanza"
          :loading="load_balanza"
          >
            <v-icon right dark> mdi-bluetooth </v-icon>
            Conectar con balanza
          </v-btn>

          <div v-show="loaded_balanza" style="margin-bottom: 1rem" ref="balanzaText">
            ¡Hemos recibido tus datos! 
            Según la báscula, pesas {{ balanzaData['weight'] }} kg.
            Puedes avanzar en la encuesta.
          </div>
        </div>

        <div v-else class="if-not-balanza">
          <v-slider
            v-model="weight.val"
            :label="weight.label"
            :thumb-color="weight.color"
            color="var(--main-orange)"
            track-color="var(--main-orange-pale)"
            thumb-label="always"
            max="150"
            min="40"
            style="margin-bottom: 10px; margin-top: 40px"
            required
          ></v-slider>
          <BodyType :body_type="body_type" @changeBodyType="body_type = $event">
          </BodyType>
        </div>
        <v-btn color="var(--main-orange)" @click="e6 = 3"> Continuar </v-btn>
      </v-stepper-content>

      <v-stepper-step :complete="e6 > 3" step="3" color="var(--main-orange)" editable>
        Actividad física y objetivos nutricionales
        <small style="margin-top: 3px">
          Saber cuánto te mueves y cuál es tu objetivo nos ayudará a adaptar el
          plan a tus necesidades</small
        >
      </v-stepper-step>

      <v-stepper-content step="3">
        <v-select
          v-model="selected_activity"
          :items="activity_types"
          label="Cuánta actividad física realizas?"
          solo
        ></v-select>

        <v-select
          v-model="objective"
          :items="objectives"
          label="Selecciona tu objetivo nutricional"
          solo
        ></v-select>
        <v-btn
          color="var(--main-orange)"
          @click="
            e6 = 4;
            created();
          "
        >
          Continuar
        </v-btn>
      </v-stepper-content>

      <v-stepper-step step="4"  color="var(--main-orange)" editable @click="created()">
        Tus preferencias de comida
        <small style="margin-top: 3px">
          Nos gustaría además recomendarte comida que te guste... aunque a veces
          hay que hacer algún sacrificio ;)</small
        >
      </v-stepper-step>
      <v-stepper-content step="4">
        <div id="completed_tinder">
          <v-snackbar
            v-model="snackbar"
            :timeout="snackbar_timeout"
            absolute
            centered
            color="var(--main-orange);"
          >
            <b>Preferencias de comida completadas</b>
            <template v-slot:action="{ attrs }">
              <v-btn
                color="var(--main-orange)"
                text
                v-bind="attrs"
                @click="snackbar = false"
              >
                Close
              </v-btn>
            </template>
          </v-snackbar>
        </div>

        <div id="gustos">
          <v-dialog
            v-model="dialog"
            transition="dialog-top-transition"
            max-width="600"
          >
            <template>
              <v-card>
                <v-toolbar style="background-color: var(--main-green)" dark>
                  <h1 class="title">
                    Ayuda: cuestionario de gustos
                  </h1></v-toolbar
                >
                <v-card-text style="margin-top: 3px">
                  Para conocer mejor tus gustos, hemos planteado esta pequeña
                  encuesta simulando el estilo de Tinder, pulsa
                  <b>me gusta</b> para recetas que te gusten,
                  <b> no me gusta</b> para aquellas que no te gusten y utiliza
                  el <b>"super-like"</b> para aquellas que te encanten. Si te
                  equivocas en alguna elección, puedes volver atrás y corregirla
                  :)
                </v-card-text>
                <v-card-actions class="justify-end">
                  <v-btn text @click="dialog = false">Close</v-btn>
                </v-card-actions>
              </v-card>
            </template>
          </v-dialog>

          <Tinder
            ref="tinder"
            key-name="id"
            :queue.sync="queue"
            :offset-y="10"
            @submit="onSubmit"
          >
            <template slot-scope="scope">
              <div
                class="pic"
                :style="{
                  'background-image': `url(${scope.data.id})`,
                }"
              />
            </template>
            <img
              class="like-pointer"
              slot="like"
              src="../assets/like-txt_tinder.png"
            />
            <img
              class="nope-pointer"
              slot="nope"
              src="../assets/nope-txt_tinder.png"
            />
            <img
              class="super-pointer"
              slot="super"
              src="../assets/super-txt_tinder.png"
            />
            <img
              class="rewind-pointer"
              slot="rewind"
              src="../assets/rewind-txt_tinder.png"
            />
          </Tinder>
          <br />
          <div class="btns">
            <img src="../assets/rewind_tinder.png" @click="decide('rewind')" />
            <img src="../assets/nope_tinder.png" @click="decide('nope')" />
            <img
              src="../assets/super-like_tinder.png"
              @click="decide('super')"
            />
            <img src="../assets/like_tinder.png" @click="decide('like')" />
            <img src="../assets/help_tinder.png" @click="decide('help')" />
          </div>
        </div>
        <br />
        <br />

        <DialogUserCreationSuccess v-model="showDialogUserSuccess" />

        <p color="success" class="btn btn-1" @click="createUser">
          <svg>
            <rect x="0" y="0" fill="none" width="100%" height="100%" />
          </svg>
          Crear nuevo usuario
        </p>
      </v-stepper-content>
    </v-stepper>
  </v-container>
</template>

<script>
import BodyType from "./BodyType.vue";
import Ingredients from "./Ingredients.vue";
import axios from "axios";
import VueSwing from "vue-swing";
import Tinder from "vue-tinder";
import DialogUserCreationSuccess from "./DialogUserCreationSuccess.vue";

export default {
  components: {
    BodyType,
    Ingredients,
    VueSwing,
    Tinder,
    DialogUserCreationSuccess,
  },
  data: () => ({
    source: {
      "./static/pollo_verduras_arroz.jpeg": ["Pollo", "Verduras", "Arroz"],
      "./static/pescado_patatas.jpeg": ["Pescado", "Patatas"],

      "./static/carne_legumbres.jpeg": ["Carne", "Legumbres"],
      "./static/pescado_arroz_verduras.jpeg": ["Pescado", "Arroz", "Verduras"],

      "./static/carne_pasta.jpeg": ["Carne", "Pasta"],
      "./static/pescado_patatas.jpeg": ["Pescado", "Patatas"],
      "./static/carne_patatas.jpeg": ["Carne", "Patatas"],

      "./static/verduras_huevos.jpeg": ["Verduras", "Huevos"],
      "./static/pollo_arroz.jpeg": ["Pollo", "Arroz"],
      "./static/legumbres.jpeg": ["Legumbres"],
      "./static/verduras_pasta.jpeg": ["Verduras", "Pasta"],

      "./static/pescado_legumbres.jpeg": ["Pescado", "Legumbres"],
      "./static/pollo_pasta.jpeg": ["Pollo", "Pasta"],
      "./static/huevos_patatas.jpeg": ["Huevos", "Patatas"],
      "./static/carne_arroz.jpeg": ["Carne", "Arroz"],
      "./static/verduras_patatas.jpeg": ["Verduras","Patatas"]
    },
    showDialogUserSuccess: false,
    queue: [],
    offset: 0,
    load_balanza: false,
    loaded_balanza : false,
    history: [],
    choices: [],
    id: "",
    last_clicked: "",
    snackbar: false,
    snackbar_timeout: 1500,
    dialog: false,
    e6: 1,
    sex_options: ["Hombre", "Mujer"],
    activity_types: [
      "1. Sedentario (nada de ejercicio / trabajo de oficina)",
      "2. Ligeramente activo (ejercicio ligero de 1 a 3 días semanales)",
      "3. Moderadamente activo (ejercicio moderado de 3 a 5 días semanales)",
      "4. Muy activo (ejercicio casi diario intenso)",
      "5. Extremadamente activo (ejercio diario muy intenso, 2 entrenamientos diarios)",
    ],
    username: "",
    name: "",
    balanzaData: {
      weight: 0
    },
    using_scale: false,
    objectives: ["Perder peso", "Mantener", "Ganar peso"],
    height: { label: "Altura (cm)", val: 165, color: "red" },
    weight: { label: "Peso (kg)", val: 65, color: "blue" },
    body_type: "default",
    age: "",
    alpha: 0.70,
    selected_activity: "",
    objective: "",
    selected_sex: "",
    mi_scale_data: {
      weight: 63.95,
      weight_unit: "kg",
      bmi: 28.42,
      basal_metabolism: 1264.14,
      visceral_fat: 8.02,
      lean_body_mass: 48.59,
      body_fat: 40.94,
      water: 42.17,
      bone_mass: 2.36,
      muscle_mass: 35.41,
      protein: 13.2,
      body_type: "Overweight",
      metabolic_age: 48,
    },
  }),

  methods: {
    toggle_using_scale() {
      if (this.using_scale) {
        this.using_scale = false;
      } else {
        this.using_scale = true;
      }
    },
    getBalanza(){
      this.load_balanza = true
      const path = this.$store.getters.baseUrl + "/balanza"
      var altura = this.height
      console.log("BALANZA")
      axios
      .post(path, altura)
      .then((res) =>{
        this.balanzaData = res.data//JSON.stringify(res.data,null,2)
        console.log(this.balanzaData)
        var successText = "¡Hemos recibido tus datos! "+ 
            "Según nuestra báscula, pesas " + this.balanzaData['weight'].toFixed(2) + " kg. " +
            "Puedes avanzar en la encuesta."
        this.$refs.balanzaText.innerText = successText;
        
        this.loaded_balanza = true
        this.load_balanza = false
      })
      .catch((err) =>{
        var errorText = "Parece que hay algun error... ¡Prueba de nuevo!"
        this.$refs.balanzaText.innerText = errorText

        this.loaded_balanza = true
        this.load_balanza = false
        console.error(error)
      });
    },
    setUser(userData) {
      // FUNCION a introducir en el código de usuarios!
      console.log(userData);
      this.$store.commit("setUserData", {
        userdata: userData,
      });
    },
    createUser() {
      var preferences = [];
      var categories = Object.values(this.source);
      for (var i = 0; i < this.history.length; i++) {
        preferences.push({
          categories: categories[i],
          rating: this.choices[i],
        });
      }

      var new_user = {
        username: this.username,
        name: this.name,
        age: parseInt(this.age, 10),
        sex: this.selected_sex,
        weight: this.weight.val,
        height: this.height.val,
        alpha: this.alpha,
        body_type: this.body_type,
        activity_level: this.activity_types.indexOf(this.selected_activity) + 1,
        objective: this.objectives.indexOf(this.objective),
        using_scale: this.using_scale,
        scale_data: this.balanzaData,
        preferences: preferences,
      };
      console.log(new_user);

      axios({
        baseURL: this.$store.getters.baseUrl,
        url: "/newUser",
        method: "post",
        data: new_user,
      })
        .then((res) => {
          console.log(res.data);
          this.setUser(res.data);
          this.showDialogUserSuccess = true;
        })
        .catch((err) => {
          console.log(err);
        });
    },
    created() {
      console.log("Hola");
      this.mock();
    },
    hideTinder() {
      var x = document.getElementById("gustos");

      if (x.style.display === "none") {
        x.style.display = "block";
      } else {
        x.style.display = "none";
        this.snackbar = true;
      }

      for (var i = 0; i < this.choices.length; i++) {
        console.log(Object.values(this.source)[i] + ":" + this.choices[i]);
      }
    },
    mock(count = Object.values(this.source).length, append = true) {
      const list = [];
      for (let i = 0; i < count; i++) {
        list.push({ id: Object.keys(this.source)[this.offset] });

        this.offset++;
      }
      if (append) {
        this.queue = this.queue.concat(list);
      } else {
        this.queue.unshift(...list);
      }
    },
    onSubmit({ item }) {
      if (this.queue.length == 0) {
        this.hideTinder();
      }

      this.history.push(item);
      this.choices.push(this.last_clicked);
      console.log("History length:", this.history.length);
      console.log("Queue length:", this.queue.length);
    },
    async decide(choice) {
      if (choice === "rewind") {
        if (this.history.length) {
          this.$refs.tinder.rewind([this.history.pop()]);
          this.choices.pop();
        }
      } else if (choice === "help") {
        this.dialog = true;
      } else {
        if (choice === "like") {
          this.last_clicked = "like";
        } else if (choice === "nope") {
          this.last_clicked = "nope";
        } else if (choice === "super") {
          this.last_clicked = "super";
        }
        this.$refs.tinder.decide(choice);
      }
    },
  },
};
</script>


<style scoped lang="scss">

.step{
  $stepper-elevation:0;
}
.formulario {
  margin-top: 1cm;
  background-color: var(--main-green);
  margin: auto;
  width: 95%;
  border: 2px solid var(--main-green);
  padding: 10px;
  border-radius: 25px;
  justify-content: flex-end;
  text-align: left;
  line-height: 300px;
}
input {
  height: 20px;
  flex: 0 0 200px;
  margin-left: 10px;
}

.checkbox-form {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.checkbox-form .answers {
  display: flex;
  flex-direction: column;
  align-items: left;
  width: 100%;
}

.checkbox-form label {
  margin-left: 1em;
}

.checkbox-form .item {
  display: block;
  position: relative;
  padding-left: 35px;
  margin-bottom: 12px;
  cursor: pointer;
  font-size: 1em;
  height: 25px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  display: flex;
  align-items: center;
}

.checkbox-form .item input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

.checkbox-form .checkmark {
  position: absolute;
  top: 1;
  left: 0;
  border-radius: 45%;
  height: 20px;
  width: 20px;
  background-color: white;
}

.checkbox-form .item input:checked ~ .checkmark {
  background-color: var(--main-orange);
}

.checkbox-form .checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.checkbox-form .item input:checked ~ .checkmark:after {
  display: block;
}

.checkbox-form .item .checkmark:after {
  left: 7px;
  top: 4px;
  width: 6px;
  height: 10px;
  border: solid white;
  border-width: 0 3px 3px 0;
  -webkit-transform: rotate(45deg);
  -ms-transform: rotate(45deg);
  transform: rotate(45deg);
}

.card {
  margin-top: 30 px;
  align-items: center;
  background-color: rgb(232, 254, 202);
  border-radius: 20px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  display: flex;
  font-size: 22px;
  height: 300px;
  justify-content: center;
  left: calc(50% - 100px);
  position: absolute;
  width: 200px;
}

#gustos .vue-tinder {
  left: 0;
  right: 0;
  bottom: 30px;
  margin: auto;
  height: 370px;
  display: flex;
  margin-top: 2%;
  align-items: center;
  justify-content: center;
  min-width: 70%;
  max-width: 355px;
}

.nope-pointer,
.like-pointer {
  position: absolute;
  z-index: 1;
  top: 20px;
  width: 64px;
  height: 64px;
}

.nope-pointer {
  right: 10px;
}

.like-pointer {
  left: 10px;
}

.super-pointer {
  position: absolute;
  z-index: 1;
  bottom: 80px;
  left: 0;
  right: 0;
  margin: auto;
  width: 112px;
  height: 78px;
}

.rewind-pointer {
  position: absolute;
  z-index: 1;
  top: 20px;
  right: 10px;
  width: 112px;
  height: 78px;
}

.pic {
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
}

.btns {
  left: 0;
  right: 0;
  bottom: 30px;
  margin: auto;
  height: 65px;
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 300px;
  max-width: 355px;
}

.btns img {
  margin-bottom: 4%;

  margin-right: 12px;
  box-shadow: 0 4px 9px rgba(0, 0, 0, 0.15);
  border-radius: 50%;
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
}

.btns img:nth-child(2n + 1) {
  width: 53px;
}

.btns img:nth-child(2n) {
  width: 65px;
}

.btns img:nth-last-child(1) {
  margin-right: 0;
}

@import url(https://fonts.googleapis.com/css?family=Roboto:400,100,900);

//colors
$red: #e1332d;
$white: #fff;

//base styles

* {
  box-sizing: inherit;
  transition-property: all;
  transition-duration: 0.6s;
  transition-timing-function: ease;
}

p {
  background: rgba($white, 0);
  margin: auto;
  line-height: 1.4;
  padding: 0.25em;
  text-decoration: none;
}
//default button
.btn {
  color: var(--main-orange);
  cursor: pointer;
  // display: block;
  font-size: 16px;
  font-weight: 400;
  line-height: 45px;
  max-width: 250px;
  position: relative;
  text-decoration: none;
  text-transform: uppercase;
  width: 100%;
}

/////////////////////////////////
//button one
///////////////////////////////
.btn-1 {
  background: white;
  font-weight: 500;

  svg {
    height: 45px;
    left: 0;
    position: absolute;
    top: 0;
    width: 100%;
  }

  rect {
    fill: none;
    stroke: var(--main-orange);
    stroke-width: 2;
    stroke-dasharray: 422, 0;
    transition: all 0.35s linear;
  }
}

.btn-1:hover {
  font-weight: 900;
  letter-spacing: 1px;
  color: var(--main-orange);

  rect {
    stroke-width: 5;
    stroke-dasharray: 15, 410;
    stroke-dashoffset: 48;
    transition: all 1.35s cubic-bezier(0.19, 1, 0.22, 1);
  }
}

////////////////////////////
//button two
//////////////////////////
.btn-2 {
  letter-spacing: 0;
}

.btn-2:hover,
.btn-2:active {
  letter-spacing: 5px;
}

.btn-2:after,
.btn-2:before {
  backface-visibility: hidden;
  border: 1px solid rgba(#fff, 0);
  bottom: 0px;
  content: " ";
  display: block;
  margin: 0 auto;
  position: relative;
  transition: all 280ms ease-in-out;
  width: 0;
}

.btn-2:hover:after,
.btn-2:hover:before {
  backface-visibility: hidden;
  border-color: #fff;
  transition: width 350ms ease-in-out;
  width: 70%;
}

.btn-2:hover:before {
  bottom: auto;
  top: 0;
  width: 70%;
}

/////////////////////////////
//button -3
///////////////////////////
.btn-3 {
  background: lighten($red, 3%);
  border: 1px solid darken($red, 4%);
  box-shadow: 0px 2px 0 darken($red, 5%), 2px 4px 6px darken($red, 2%);
  font-weight: 900;
  letter-spacing: 1px;
  transition: all 150ms linear;
}

.btn-3:hover {
  background: darken($red, 1.5%);
  border: 1px solid rgba(#000, 0.05);
  box-shadow: 1px 1px 2px rgba(#fff, 0.2);
  color: lighten($red, 18%);
  text-decoration: none;
  text-shadow: -1px -1px 0 darken($red, 9.5%);
  transition: all 250ms linear;
}
</style>