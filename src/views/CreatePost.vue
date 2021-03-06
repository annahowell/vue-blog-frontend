<template>
  <v-container>
    <div v-if="loaded">
      <v-row>
        <h4 class="display-1 mb-9">Create post</h4>
      </v-row>

      <v-row>
        <div class="v-messages theme--light mb-0 pl-1">Tags</div>
      </v-row>

      <tags :postTags="[]" :tags="tags" @setFormtags="setFormtags"/>

      <v-row>
        <v-card flat width="100%">
          <v-form
            v-model="titleAppearsValid"
            @submit.prevent="submitPost"
            ref="form"
            class="pa-0 pt-0"
          >
            <v-card-text class="pt-0 pb-0 py-2">
              <v-row class="mb-9">
                <v-text-field
                  :rules='[rules.required, rules.min8, rules.max255]'
                  v-model="form.title"
                  label="Title"
                  placeholder=" "
                />
              </v-row>

              <v-row class="create-post">
                <div class="v-messages theme--light mb-1" :style="bodyLabel">Body</div>

                <tiptap-vuetify
                  v-model="form.body"
                  :rules='[rules.required]'
                  :extensions="extensions"
                  :card-props="{ flat: true }"
                  :toolbar-attributes="{ color: 'grey lighten-4' }"
                  @focus="handleBodyState(true)"
                  @blur="handleBodyState(false)"
                  style="width:100%"
                  placeholder=" "
                />

                <div :style="{width: '100%', borderBottom: '1px solid ' + bodyLine.color}">
                  <div :style="bodyLine"/>
                </div>
              </v-row>

              <v-row style="min-height:26px">
                <v-slide-y-transition>
                  <div v-if="!bodyAppearsValid" class="v-messages error--text">Required</div>
                </v-slide-y-transition>
              </v-row>

              <v-row class="mb-3">
                <v-alert
                  v-model="postErrorVisible"
                  transition="fade-transition"
                  type="error"
                  class="caption mb-0"
                  width="100%"
                  dense text
                >{{postErrorMessage}}</v-alert>
              </v-row>
            </v-card-text>

            <v-card-actions class="pa-0 pl-2">
              <v-spacer/>

              <v-btn
                :disabled="!titleAppearsValid || this.form.body == '<p></p>'"
                type="submit"
                color="primary"
                text
              >Post</v-btn>
            </v-card-actions>
          </v-form>
        </v-card>
      </v-row>
    </div>

    <div v-else>
      <v-skeleton-loader :class="loginData.isAdmin ? 'mt-2 mb-16' : 'mt-2 mb-7'" max-width="50%" type="heading"/>

      <v-skeleton-loader class="mx-0 pt-9 mb-16 pb-8" width="100%" type="table-thead"/>
    </div>
  </v-container>
</template>

<script>
import { mapState } from 'vuex'
import { TiptapVuetify, Heading, Bold, Italic, Strike, Underline, Code, Paragraph, BulletList, OrderedList, ListItem, Link, Blockquote, HardBreak, History } from 'tiptap-vuetify'
import Tags from '@/components/Tags'

export default {
  name: 'CreatePost',

  components: {
    TiptapVuetify,
    Tags
  },

  data () {
    return {
      showManageTagDialog: false,
      bodyHasNotBeenEnteredOrClicked: true,
      titleAppearsValid: false,

      postErrorVisible: false,
      postErrorMessage: '',
      errorColor: this.$vuetify.theme.themes.light.error,
      primaryColor: this.$vuetify.theme.themes.light.primary,

      form: {
        title: '',
        body: '<p></p>',
        tags: []
      },

      bodyLabel: {
        color: '',
        animation: ''
      },
      bodyLine: {
        width: '100%',
        color: 'rgba(0, 0, 0, .42)',
        transform: 'scaleX(0)',
        borderStyle: 'solid',
        borderWidth: 'thin 0 0',
        transition: '.3s cubic-bezier(.25,.8,.5,1)',
      },

      rules: {
        required: val => !!val || 'Required',
        min8: val => val.length > 7 || 'Minumum of 8 charcters',
        max255: val => val.length < 256 || 'Maximum of 255 characters'
      },

      extensions: [
        History,
        Bold,
        Underline,
        Italic,
        Strike,
        BulletList,
        ListItem,
        OrderedList,
        [Heading, {
          options: {
            levels: [1, 2, 3]
          }
        }],
        Blockquote,
        Code,
        Link,
        Paragraph,
        HardBreak,
      ]
    }
  },

  created () {
    this.$store.dispatch("getAllTags")
  },

  computed: {
    loaded () {
      return this.tags !== null
    },

    bodyAppearsValid () {
      return this.form.body != '<p></p>' || this.bodyHasNotBeenEnteredOrClicked
    },

    ...mapState([
      'loginData',
      'tags'
    ])
  },

  methods: {
    setFormtags (selectedTags) {
      this.form.tags = selectedTags
    },

    handleBodyState (focused) {
      if (focused) {
        this.bodyLabel.color    = this.bodyAppearsValid ? this.primaryColor : this.errorColor
        this.bodyLine.color     = this.bodyLabel.color
        this.bodyLine.transform = 'scaleX(1)'
      } else {
        this.bodyHasNotBeenEnteredOrClicked = false
        this.bodyLabel.color    = this.bodyAppearsValid ? '' : this.errorColor
        this.bodyLine.color     = this.bodyAppearsValid ? 'rgba(0, 0, 0, .42)' : this.errorColor
        this.bodyLine.transform = 'scaleX(0)'
      }

      this.bodyLabel.animation = this.bodyAppearsValid ? '' : 'v-shake .6s cubic-bezier(.25,.8,.5,1)'
    },

    submitPost () {
      if (this.titleAppearsValid && this.bodyAppearsValid) {
        let postData = {
          title: this.form.title,
          body: this.form.body,
          tags: this.form.tags
        }

        this.$store.dispatch("postPost", {vm: this, postData: postData})
        .catch(error => {
          if (error.response.status === 422) {
            this.postErrorMessage = Object.values(error.response.data.errors)[0][0]
            this.postErrorVisible = true
          }
        })
      }
    }
  },

  watch: {
    'form.body' () {
      this.bodyHasNotBeenEnteredOrClicked = false
      // true it's focused as if it weren't the body wouldn't change
      this.handleBodyState(true)
    }
  }
}
</script>

<style>
.create-post .tiptap-vuetify-editor__content {
  padding: 0;
}

.create-post .ProseMirror {
  margin: 0!important;
  min-height: 400px!important;
}

.v-skeleton-loader__table-thead {
  padding: 8px 16px;
}

.v-skeleton-loader__table-thead > div {
  max-width: 15%!important;
  height: 32px;
  border-radius: 16px;
}
</style>
