<template>
  <div v-if="ad">
    <a
      v-if="hasPromoted"
      class="d-block"
      style="max-width: 640px;"
      v-bind="attrs"
      @click="onClick"
    >
      <promoted-base
        v-bind="$attrs"
        class="v-vuetify--promoted"
        density="compact"
        max-width="640"
        outlined
      >
        <v-img
          :src="background"
          class="flex-1-1-auto rounded"
          max-height="56"
          cover
        >
          <div class="d-flex align-center fill-height">
            <v-img
              :alt="`Link to ${ad.title}`"
              :src="logo"
              class="mx-2"
              contain
              height="56"
              max-width="56"
            />

            <app-markdown
              v-if="description"
              :content="description"
              class="text-subtitle-2 text-sm-h6 font-weight-light text-white"
            />
          </div>
        </v-img>
      </promoted-base>
    </a>

    <vuetify v-else />
  </div>
</template>

<script setup>
  // Components
  import PromotedBase from './Base.vue'
  import Vuetify from './Vuetify.vue'

  // Composables
  import { createAdProps, useAd } from '@/composables/ad'
  import { useGtag } from 'vue-gtag-next'

  // Utilities
  import { computed } from 'vue'

  const props = defineProps({
    ...createAdProps(),

    medium: {
      type: String,
      default: 'promoted',
    },
  })

  const { ad, attrs } = useAd(props)
  const { event } = useGtag()

  const background = computed(() => ad.value?.metadata?.images?.background?.url)
  const hasPromoted = computed(() => {
    return (
      ad.value?.metadata?.description_short &&
      background.value
    )
  })

  const description = computed(() => ad.value?.metadata?.description_short || ad.value?.metadata?.description)
  const logo = computed(() => {
    if (props.medium === 'promoted') {
      return ad.value?.metadata?.images?.preview?.url || ad.value?.metadata?.images?.logo?.url
    }

    return ad.value?.metadata?.images?.logo?.url
  })

  function onClick () {
    const slug = ad.value?.slug

    if (!slug) return

    event('click', {
      event_category: 'vuetifys',
      event_label: slug,
      value: 'promoted',
    })
  }
</script>

<script>
  export default {
    inheritAttrs: false,
  }
</script>

<style lang="sass">
  .v-vuetify--promoted
    p
      line-height: 1.1

    .v-markdown p strong
      font-weight: 700
</style>
