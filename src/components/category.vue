<template>
  <section
    :class="{
      'emoji-mart-category': true,
      'emoji-mart-no-results': !hasResults,
    }"
    :aria-label="i18n.categories[id]"
    v-if="isVisible && (isSearch || hasResults)"
  >
    <div class="emoji-mart-category-label">
      <h3 class="emoji-mart-category-label">{{ i18n.categories[id] }}</h3>
    </div>

    <template v-for="{ emojiObject, emojiView } in emojiObjects">
      <button
        v-if="emojiView.canRender"
        :aria-label="emojiView.ariaLabel"
        role="option"
        aria-selected="false"
        aria-posinset="1"
        aria-setsize="1812"
        type="button"
        :data-title="emojiObject.short_name"
        :key="emojiObject.id"
        :title="emojiView.title"
        class="emoji-mart-emoji"
        :class="activeClass(emojiObject)"
        @mouseenter="onEnter($event, emojiObject, emojiView)"
        @mouseleave="onLeave($event, emojiObject, emojiView)"
        @click="emojiProps.onClick(emojiView.getEmoji())"
      >
        <img
          v-if="showExternalEmoji && emojiObject._data.has_img_external"
          :src="showExternalAll ? emojiObject._data.externalUrl : transparentPixel"
          :class="emojiView.cssClass"
          :style="emojiView.cssStyle"
          class="emoji-type-external"
          @load="onLoad"
          @error="onError"
          loading="lazy"
        />
        <span v-else :class="emojiView.cssClass" :style="emojiView.cssStyle">{{
          emojiView.content
        }}</span>
      </button>
    </template>

    <div v-if="!hasResults">
      <emoji
        :data="data"
        emoji="sleuth_or_spy"
        :native="emojiProps.native"
        :skin="emojiProps.skin"
        :set="emojiProps.set"
      />
      <div class="emoji-mart-no-results-label">{{ i18n.notfound }}</div>
    </div>
  </section>
</template>

<script>
import { EmojiView, transparentPixel } from '../utils/emoji-data'
import Emoji from './Emoji.vue'

export default {
  props: {
    data: {
      type: Object,
      required: true,
    },
    i18n: {
      type: Object,
      required: true,
    },
    id: {
      type: String,
      required: true,
    },
    name: {
      type: String,
      required: true,
    },
    emojis: {
      type: Array,
    },
    emojiProps: {
      type: Object,
      required: true,
    },
    externalPicker: {
      type: String,
      required: true,
    }
  },
  data() {
    return {
      transparentPixel,
    }
  },
  methods: {
    onEnter(event, emojiObject, emojiView) {
      this.emojiProps.onEnter(emojiView.getEmoji())
      if (this.showExternalHover) {
        const el = event.target.querySelector('img.emoji-type-external');
        if (el) {
          el.src = emojiObject._data.externalUrl;
          el.style.backgroundImage = null;
        }
      }
    },
    onLeave(event, emojiObject, emojiView) {
      this.emojiProps.onLeave(emojiView.getEmoji())
      if (this.showExternalHover) {
        const el = event.target.querySelector('img.emoji-type-external');
        if (el) {
          el.src = this.transparentPixel;
          el.style.backgroundImage = null;
        }
      }
    },
    onLoad(event){
      if (event.target.src !== this.transparentPixel) {
        event.target.style.backgroundImage = 'unset';
      }
    },
    onError(event) {
      event.target.src = this.transparentPixel;
      event.target.style.backgroundImage = null;
    },
    activeClass: function(emojiObject) {
      if (!this.emojiProps.selectedEmoji) {
        return ''
      }
      if (!this.emojiProps.selectedEmojiCategory) {
        return ''
      }
      if (
        this.emojiProps.selectedEmoji.id == emojiObject.id &&
        this.emojiProps.selectedEmojiCategory.id == this.id
      ) {
        return 'emoji-mart-emoji-selected'
      }
      return ''
    },
  },
  computed: {
    showExternalAll() {
      return this.externalPicker === 'all'
    },
    showExternalHover() {
      return this.externalPicker === 'hover'
    },
    showExternalEmoji() {
      return this.emojiProps.externalEnabled && (this.showExternalAll || this.showExternalHover)
    },
    isVisible() {
      return !!this.emojis
    },
    isSearch() {
      return this.name == 'Search'
    },
    hasResults() {
      return this.emojis.length > 0
    },
    emojiObjects() {
      return this.emojis.map((emoji) => {
        let emojiObject = emoji
        let emojiView = new EmojiView(
          emoji,
          this.emojiProps.skin,
          this.emojiProps.set,
          this.emojiProps.native,
          this.emojiProps.fallback,
          this.emojiProps.emojiTooltip,
          this.emojiProps.emojiSize,
          this.emojiProps.externalEnabled,
        )
        return { emojiObject, emojiView }
      })
    },
  },
  components: {
    Emoji,
  },
}
</script>
