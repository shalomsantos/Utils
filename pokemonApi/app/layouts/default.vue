<template>
<v-layout ref="app" class="rounded rounded-md border">
    <v-app-bar color="amber-lighten-1" name="app-bar" density="compact">
      <v-btn class="mx-auto">PokéDex</v-btn>
    </v-app-bar>

    <v-main class="bg-grey-lighten-4" style="min-height: 99.5vh;">
      <v-sheet class="bg-transparent pa-1">
        <slot />
      </v-sheet>
    </v-main>
  </v-layout>  
</template>

<script setup>
</script>

<style scoped>

</style>