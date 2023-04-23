<template>
  <div id="app">
    <h1>Witaj w systemie do zapisów na zajęcia</h1>

    <div v-if="authenticatedUsername">
      <UserPanel :username="authenticatedUsername" @logout="logMeOut()"></UserPanel>
      <MeetingsPage :username="authenticatedUsername"></MeetingsPage>
    </div>

    <div v-else>
      <button @click="registering = false"
              :class="registering ? 'button-outline' : ''">Logowanie</button>
      <button @click="registering = true"
              :class="!registering ? 'button-outline' : ''">Rejestracja</button>

      <div v-if = "message" :class = "message.includes('istnieje') ? 'alert alert-red' : 'alert'" > {{ message }}</div>

      <LoginForm v-if = "!registering" @login="(user) => logMeIn(user)"></LoginForm>
      <LoginForm v-else @login="(user) => register(user)" button-label="Załóż konto"></LoginForm>

    </div>
  </div>
</template>

<script>
import "milligram";
import LoginForm from "./LoginForm";
import UserPanel from "./UserPanel";
import MeetingsPage from "./meetings/MeetingsPage";
import axios from "axios";

export default {
  components: {LoginForm, MeetingsPage, UserPanel},
  data() {
    return {
      registering: false,
      authenticatedUsername: '',
      message: '',
      userCreated: false
    }
  },
  methods: {
    logMeIn(user) {
      axios.post('/api/tokens', user)
          .then(response => {
            const token = response.data.token;
            axios.defaults.headers.common['Authorization'] = 'Bearer ' + token;
            this.authenticatedUsername = user.login;
            axios.get('/api/meetings').then(response => console.log(response.data));
          })
          .catch(response => {
            this.message = 'Nie udało się zalogować!';
          });

    },
    logMeOut() {
      this.authenticatedUsername = '';
      delete axios.defaults.headers.common.Authorization;
    },

    register(user) {
      axios.post('/api/participants', user)
          .then(response => {
            // alert("Konto zostało pomyślnie założone!");
            this.userCreated = true;
            this.message = 'Konto zaostał ozałożone pomyślnie!'
          })
          .catch(response => {
            // alert("Konto o tym loginie już istnieje, spróbuj podać inny!");
            this.message = 'Konto o podanym loginie już istnieje!'
          });
    }
  }
}
</script>

<style>
#app {
  max-width: 1000px;
  margin: 0 auto;
}

.alert {
  border: 2px;
  background: lightgreen;
  border: green;
  text-align: center;
  padding: 10px;
  margin: 10px;
}

.alert-red {
  background: indianred;
  border-color: red;
}
</style>
