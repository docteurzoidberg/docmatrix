<template lang="pug">
  v-app(dark)
    v-app-bar(
      app
      color="secondary"
      dark
    )
      .d-flex.align-center
        v-img(
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        )
        | DOCIXEL
      v-spacer
      .flagcontainer
        v-select(
          v-model="selectedLanguage"
          :items="languages"
          v-show="isLangSelectOpen"
          v-on:input="langSelectClose",
          v-on:change="langSelectClose"
          width="40"
          ref="langSelect"
          :menu-props="{value: isLangSelectOpen, closeOnClick: true, closeOnContentClick: true}"
        )
          template(slot="item", slot-scope="data")
            flag(:iso="data.item")
            | {{data.item}}
        .flag(v-show="!isLangSelectOpen", v-on:click="langSelectOpen")
          flag(:iso="this.selectedLanguage")
    v-main
      router-view

</template>

<script>

export default {
  name: 'App',

  components: {
  },

  data: () => ({
    selectedLanguage: 'fr',
    isLangSelectOpen: false,
    languages: [
      'us','fr'
    ]

  }),
  methods: {
    langSelectOpen() {
      this.isLangSelectOpen = true;
      this.$nextTick(() => this.$refs.langSelect.focus())
    },
    langSelectClose() {
      this.isLangSelectOpen = false;
      console.log("close");
    }
  }
};
</script>

<style scoped>
.flag  {
  font-size: 32px;
}
.flagcontainer {
  max-width: 60px;
}
</style>