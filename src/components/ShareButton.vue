<template>
  <v-btn
    v-if="!$vuetify.display.smAndUp"
    class="ml-2"
    size="small"
    density="comfortable"
    color="teal"
    icon="mdi-share-variant"
    :title="$t('ShareButton.Title')"
    @click="shareViaWebShare">
  </v-btn>
  <v-btn
    v-else
    class="ml-2"
    size="small"
    rounded="xl"
    color="teal"
    prepend-icon="mdi-share-variant"
    :title="$t('ShareButton.Title')"
    @click="shareViaWebShare">
    {{ $t('ShareButton.Title') }}
  </v-btn>

  <v-snackbar
    v-model="shareLinkCopySuccessMessage"
    color="success"
    :timeout="2000">
    {{ $t('ShareButton.LinkCopySuccess') }}
  </v-snackbar>
</template>

<script>
import constants from '../constants'

export default {
  props: {
    'overrideUrl': null,
  },
  data() {
    return {
      APP_NAME: constants.APP_NAME,
      shareLinkCopySuccessMessage: false
    }
  },
  methods: {
    shareViaWebShare() {
      let URL = `${import.meta.env.VITE_OPEN_PRICES_APP_URL}${this.overrideUrl ? this.overrideUrl : this.$route.href}`
      if (navigator.share) {
        navigator.share({
          title: this.APP_NAME,
          url: URL
        })
      } else {
        navigator.clipboard.writeText(URL)
        this.shareLinkCopySuccessMessage = true
      }
    }
  }  
}
</script>
