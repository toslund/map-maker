<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <v-toolbar-title>Map Maker</v-toolbar-title>
      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/toslund/map-maker"
        target="_blank"
        text
      >
        <span class="mr-2">GH</span>
        <v-icon>mdi-github</v-icon>
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
