<template>
  <component
    :is="tag"
    v-if="view.canRender"
    :title="view.title"
    :aria-label="view.ariaLabel"
    :data-title="title"
    class="emoji-mart-emoji"
    @mouseenter="onMouseEnter"
    @mouseleave="onMouseLeave"
    @click="onClick"
  >
    <img
      v-if="externalEnabled && emojiObject._data.has_img_external"
      ref="externalEmoji"
      :class="view.cssClass"
      :style="view.cssStyle"
      class="emoji-type-external"
      @load="onLoad"
      @error="onError"
      loading="lazy"
    />
    <span v-else :class="view.cssClass" :style="view.cssStyle">{{
      view.content
    }}</span>
  </component>
</template>

<script>
import { EmojiProps } from '../utils/shared-props'
import { EmojiView, transparentPixel } from '../utils/emoji-data'

export default {
  props: {
    ...EmojiProps,
    data: {
      type: Object,
      required: true,
    },
    externalEnabled: {
      type: Boolean,
      default: false,
    },
  },
  emits: ["click", "mouseenter", "mouseleave"],
  computed: {
    externalUrl() {
      return this.emojiObject._data.externalUrl
    },
    view() {
      return new EmojiView(
        this.emojiObject,
        this.skin,
        this.set,
        this.native,
        this.fallback,
        this.tooltip,
        this.size,
        this.externalEnabled,
      )
    },
    sanitizedData() {
      return this.emojiObject._sanitized
    },
    title() {
      return this.tooltip ? this.emojiObject.short_name : null
    },
    emojiObject() {
      if (typeof this.emoji == 'string') {
        return this.data.findEmoji(this.emoji)
      } else {
        return this.emoji
      }
    },
  },
  watch: {
    externalUrl: 'externalUrlHandler',
  },
  mounted() {
    this.externalUrlHandler()
  },
  methods: {
    onClick() {
      this.$emit('click', this.emojiObject)
    },
    onMouseEnter() {
      this.$emit('mouseenter', this.emojiObject)
    },
    onMouseLeave() {
      this.$emit('mouseleave', this.emojiObject)
    },
    onLoad(event) {
      if (event.target.src !== transparentPixel) {
        event.target.style.backgroundImage = 'unset'
      }
    },
    onError(event) {
      event.target.src = transparentPixel
      event.target.style.backgroundImage = null
    },
    externalUrlHandler() {
      if (!this.externalUrl) {
        return;
      }
      this.$nextTick(() => {
        if (!this.$refs.externalEmoji) {
          return
        }
        const emojiEl = this.$refs.externalEmoji;
        emojiEl.src = transparentPixel
        emojiEl.style.backgroundImage = null
        this.$nextTick(() => (emojiEl.src = this.externalUrl))
      })
    }
  },
}
</script>
