<template lang="pug">
  q-layout.no-shadow(view="lHh Lpr lFf")
    q-layout-footer.no-shadow
      q-toolbar.row(color="primary")
        // q-item
          img(src="statics/guc-logo-64x64.png")
        q-toolbar-title.col
          img(src="statics/G-U-C-logo-header.png" height="21px" style="padding-right:10px;margin-bottom:-4px")
          span Github-Utopian-Contributor
          // div(slot="subtitle") Quasar v{{ $q.version }}

        q-btn-dropdown(ref="selectLanguages", v-if="hasMultipleLanguages", icon="translate", :label="currentLanguage.code", flat, dense)
          q-list(link)
            q-item(v-close-overlay, v-for="(language, index) in availableLanguages", :key="index", @click.native="setLanguage(language.code)")
              q-item-main
                q-item-tile(label) {{ language.name }}
              q-item-side(v-if="language.code === currentLanguage.code", right, icon="done", color="primary")
    q-page-container
      router-view
    // q-layout-footer.no-shadow
      q-tabs(
        @select="select"
        inverted
        position="bottom"
        align="justify"
      )
        q-tab(default slot="title" name="Config" icon="fas fa-cogs" label="Config")
        q-tab(slot="title" name="Editor" icon="fas fa-pen-nib" label="Editor")
        q-tab(slot="title" name="Preview" icon="far fa-eye" label="Preview")
</template>
<style>
  /* electron fix from https://github.com/electron/electron/issues/4420#issuecomment-242835469 */
  ::selection {
    background:rgba(255, 255, 125, 0.99);
    color:#032764;
  }
  body {
    overflow-y: scroll;
  }
</style>
<script>
import { mixin as i18nMixin } from '@/plugins/i18n'

export default {
  name: 'MainLayout',
  mixins: [i18nMixin],
  methods: {
    select (tab) {
      this.$q.notify(`Switched to: ${tab}`)
    }
  }
}
</script>
