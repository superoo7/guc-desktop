<template lang="pug">
  q-page
    q-tabs(
      align="justify"
      position="top"
    )
      q-tab(default slot="title" name="Config" icon="fas fa-cogs")
        label {{ $t('pages.interface.config.label') }}
      q-tab(slot="title" name="Edit" icon="fas fa-pen-nib")
        label {{ $t('pages.interface.edit.label') }}
      q-tab(slot="title" name="Review" icon="far fa-eye")
        label {{ $t('pages.interface.review.label') }}
      q-tab(slot="title" name="Inform" icon="far fa-question-circle")
        label {{ $t('pages.interface.inform.label') }}
      q-tab-pane.q-pa-sm.row(name="Config")
        q-item.col-md-6
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="config.steemAccount"
              :style="`background-image: url('https://steemitimages.com/u/${config.steemAccount}/avatar')!important; background-size: 42px!important; background-position: center center;`"
            )
          q-item-main
            q-input(
              v-model="config.steemAccount"
              :float-label="$t('pages.interface.config.steemAccount.label')"
              :placeholder="$t('pages.interface.config.steemAccount.hint')"
            )
        q-item.col-md-6
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="config.gitUser"
              :style="`background-image: url('https://github.com/${config.gitUser}.png')!important; background-size: 42px 42px!important;`"
            )
          q-item-main
            q-input(
              v-model="config.gitUser"
              :float-label="$t('pages.interface.config.gitUser.label')"
              :placeholder="$t('pages.interface.config.gitUser.hint')"
            )
        q-item.col-md-6
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="config.steemPostingKey"
              icon="fas fa-key"
            )
          q-item-main
            q-input(
              type="password"
              v-model="config.steemPostingKey"
              :float-label="$t('pages.interface.config.steemPostingKey.label')"
              :placeholder="$t('pages.interface.config.steemPostingKey.hint')"
            )

        q-item.col-md-6
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="config.gitRepo"
              icon="fab fa-git-square"
            )
          q-item-main
            q-input(
              v-model="config.gitRepo"
              :float-label="$t('pages.interface.config.gitRepo.label')"
              :placeholder="$t('pages.interface.config.gitRepo.hint')"
            )
        q-item.col-md-6
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="config.gitRepo"
              icon="fas fa-folder"
            )
          q-item-main
            q-input(
              v-model="config.workingDirectory"
              :float-label="$t('pages.interface.config.workingDirectory.label')"
              :placeholder="$t('pages.interface.config.workingDirectory.hint')"
            )
        q-item.col-md-6
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="config.gitRepo"
              icon="fas fa-hashtag"
            )
          q-item-main
            q-input(
              v-model="config.commitId"
              :float-label="$t('pages.interface.config.commitId.label')"
              :placeholder="$t('pages.interface.config.commitId.hint')"
            )

      q-tab-pane.q-pa-sm.row(name="Edit")
        q-item.col-md-12
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="tags"
              icon="fas fa-feather"
            )
          q-item-main
            q-field(:count="5", :max-count="5" :max="5")
              q-chips-input(
                color="secondary"
                :float-label="$t('pages.interface.edit.tags')"
                v-model="tags"
                @input="steemTag"
              )
        q-item.col-md-12
          q-item-side(color='secondary' style="margin-left:-1px")
            q-btn(
              round
              v-if="postTitle"
              icon="fas fa-dove"
            )
          q-item-main
            q-input(
              v-model="postTitle"
              :float-label="$t('pages.interface.edit.postTitle')"
              placeholder="Keep it short and simple"
            )
        // q-btn(icon="visibility" v-bind="$attrs" @click="show")
        // create-button()
        // remove-button()
        q-editor.col-md-12(
          v-model="model"
          :toolbar="[['bold', 'italic', 'underline', 'strike'],['link'],[{label: 'Sizes', icon: 'format_sizes', list: 'no-icons', options: ['p', 'code', 'h5', 'h4', 'h3', 'h2', 'h1']}]]"
        )

      q-tab-pane.q-pa-sm(name="Review")
        .row
          .col-12.full-width
            div {{ $t('pages.interface.config.steemAccount.label') }}:&nbsp;
              strong {{ config.steemAccount }}
            div {{ $t('pages.interface.edit.postTitle') }}:&nbsp;
              strong {{ postTitle }}
            div {{ $t('pages.interface.edit.tags') }}:&nbsp;
              strong {{ tags }}
            .markdownDisplay
              pre(
                v-model="model_ce"
                :contenteditable="experimental.contentEditable"
                @blur="content.body = model_ce"
              ) {{ content.header }}{{ content.body }}{{ content.footer }}
            q-checkbox(v-model="toHTML" @input="makeMarkdown(content.body, toHTML)") HTML
            span &nbsp;&nbsp;&nbsp;&nbsp;
            q-checkbox.disable(v-model="experimental.contentEditable" @input="!experimental.contentEditable")
              |EDIT
              span.experimental (danger!!!)
            br
            br

        action-button(
          :config="config"
          :content="content"
          :post-title="postTitle"
          :post-tags="tags"
        )

        git-log(:config="config")

      q-tab-pane.q-pa-sm(name="Inform")
        .row
          .col-1
          .col-10
            div
              br
            div(v-html="$t('pages.interface.inform.p1')")
              br
            div
              br
            div(v-html="$t('pages.interface.inform.p2')")
            div
              br
            div(v-html="$t('pages.interface.inform.p3')")
          .col-1
</template>

<script>
// import { throttle } from 'quasar' // TODO: throttle instant profile images to avoid bloating the console
import { lstatSync, existsSync } from 'fs'
import ActionButton from '@/components/ActionButton.vue'
import GitLog from '@/components/GitLog.vue'

const isDirExists = dirPath => existsSync(dirPath) && lstatSync(dirPath).isDirectory()
const isEmptyObject = obj => !obj ? true : (Object.keys(obj).length === 0) && obj.constructor === Object

export default {
  name: 'PageDashboard',
  components: {
    ActionButton,
    GitLog
  },

  data () {
    const experimental = JSON.parse(localStorage.getItem('experimental'))
    return {
      model: 'What\'s your story today 🙂',
      model_ce: '',
      toHTML: false,
      tags: ['utopian-io', 'development'],
      postTitle: '',
      experimental: isEmptyObject(experimental) ? {
        // DEFINE experimental feature here
        contentEditable: false
      } : experimental,
      config: JSON.parse(localStorage.getItem('config')) || {
        steemAccount: '',
        steemPostingKey: '',
        // steemPassword: '', // obselete since issue#14 is fixed
        gitUser: '',
        gitRepo: '',
        workingDirectory: '',
        commitId: ''
      },
      content: {
        header: '',
        body: '',
        footer: ''
      }
    }
  },

  meta () {
    return {
      title: 'Dashboard'
    }
  },

  watch: {
    experimental: {
      handler (val) {
        const { contentEditable, ...whitelist } = val
        localStorage.setItem('experimental', JSON.stringify(whitelist))
        // INSERT custom logic for experimental feature here
      },
      deep: true
    },
    model: {
      handler (val) {
        this.makeMarkdown(val)
      },
      immediate: true
    },
    config: {
      handler (val, old) {
        const { workingDirectory, gitUser, gitRepo, commitId } = val

        localStorage.setItem('config', JSON.stringify(val))
        if (isDirExists(workingDirectory)) process.chdir(workingDirectory)

        this.content.header = gitRepo ? '#### Repository\n' + gitRepo + '\n\n' : ''
        this.content.footer = gitUser ? '\n\n---\n#### GitHub Account\n' + `https://github.com/${gitUser}` : ''
        try { this.model = this.$git.notes.at(commitId).show() } catch {}
      },
      deep: true,
      immediate: true
    }
  },

  methods: {
    makeMarkdown (data, html) {
      if (this.toHTML === true || html === true) {
        this.content.body = this.$marked(data)
      } else {
        this.content.body = this.$turndown.turndown(data)
      }
    },
    steemTag () { // A function to lowercase, hyphenate and truncate the tags
      this.tags = this.tags.slice(0, 5)
      this.tags.forEach((tag, index) => {
        this.tags[index] = tag.toLowerCase()
          .replace(/[._— ]/g, '-')
          .replace(/[^a-z0-9-]/gi, '')
      })
    }
  }
}
</script>

<style>
  .disable {
    cursor: pointer;
    color: grey;
  }
  .experimental {
    margin-left: 5px;
    color: rgb(168, 106, 106);
    font-weight: bold;
  }
  .q-tabs-bar {
    border-bottom-width:6px!important;
    height: 6px!important;
  }
  .q-chip {
    margin:0 10px 0 0!important;
  }
  .markdownDisplay {
    border: 1px solid #333389;
    height: 350px;
    padding: 10px;
    overflow: auto;
    resize: vertical
  }
  pre {
    white-space: pre-wrap;
  }
  pre {
    white-space: pre-wrap;
  }
</style>
