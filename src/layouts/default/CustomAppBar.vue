<template>
  <v-app-bar
    color="primary"
    prominent>
    <v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer" />
    <v-toolbar-title>
      Sistema de propiedades
    </v-toolbar-title>
  </v-app-bar>

  <v-navigation-drawer
    v-model="drawer"
    location="left"
    temporary
  >
    <v-list-item
      title="Menú"
    ></v-list-item>

    <v-divider></v-divider>

    <v-list density="compact" nav>
      <v-list-item @click="$router.push('/')"
        prepend-icon="mdi-view-dashboard"
        title="Inicio"
        value="value"
      ></v-list-item>

      <v-list-item @click="$router.push('crud-personas')"
        prepend-icon="mdi-table"
        title="Tabla de personas"
        value="value"
      ></v-list-item>

      <v-list-item @click="$router.push('crud-propiedades')"
        prepend-icon="mdi-table"
        title="Tabla de propiedades"
        value="value"
      ></v-list-item>

      <v-list-item @click="$router.push('crud-propietarios')"
        prepend-icon="mdi-table"
        title="Tabla de propietarios"
        value="value"
      ></v-list-item>

      <v-list-item @click="$router.push('crud-arrendatarios')"
        prepend-icon="mdi-table"
        title="Tabla de arrendatarios"
        value="value"
      ></v-list-item>
    </v-list>
  </v-navigation-drawer>
</template>

<script setup>
  import { ref } from 'vue';

  const drawer = ref(false);
</script>
