<template lang="pug">
  .home
    v-expansion-panels(multiple, v-model="panel")
      v-expansion-panel
        v-expansion-panel-header Parametres
        v-expansion-panel-content
          v-slider(v-model="gridSize", min="8", max="64", label="Grille")
            template(v-slot:append)
              v-text-field(
                v-model="gridSize"
                class="mt-0 pt-0"
                hide-details
                single-line
                type="number"
                style="width: 60px"
                readonly
              )
      v-expansion-panel
        v-expansion-panel-header Palette
        v-expansion-panel-content //todo
      v-expansion-panel
        v-expansion-panel-header Dessin
        v-expansion-panel-content
          div
            Board3D(:gridSize="gridSize")
</template>

<script>
// @ is an alias to /src

import Board3D from '@/components/Board3D.vue'

export default {
  name: 'Home',
  components: {
    //HelloWorld
    Board3D
  },
  data() {
    return {
      panel: [0,1,2],
      gridSize: 32
    }
  }
}
</script>
