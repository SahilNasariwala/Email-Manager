<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          Email Manager
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <div class="full-height q-pa-sm">
        <router-view />        
      </div>
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'MainLayout',
})
</script>
