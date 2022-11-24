<template>
  <v-app>
    <v-app-bar app color="grey lighten-1">
      <div class="d-flex align-center">
        <v-img alt="Vuetify Logo" class="shrink mr-2" contain
          src="http://www.theredballoon.com.au/wp-content/uploads/orders/LX-Logo.png" transition="scale-transition"
          width="40" />

        <h1>Analizador Lexico</h1>
      </div>

      <v-spacer></v-spacer>

      <v-btn href="http://w3.unpocodetodo.info/utiles/regex.php" target="_blank" text>
        <span class="mr-2">ReGex</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <analizador></analizador>
    </v-main>
  </v-app>
</template>

<script>
import analizador from '@/components/analizador.vue'

export default {

  components: {
    analizador
  },
  data: () => ({
    //
  }),
};
</script>
