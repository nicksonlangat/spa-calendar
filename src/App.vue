<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <p>CRUD CAL</p>
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/nicksonlangat"
        target="_blank"
        text
      >
        <span class="mr-2">NICK'S EVENT CALENDAR</span>
        <v-icon>mdi-calendar</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>
