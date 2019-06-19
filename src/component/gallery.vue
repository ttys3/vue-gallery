<template>
  <div
    :id="id"
    class="blueimp-gallery blueimp-gallery-controls"
    :class="{'blueimp-gallery-carousel': carousel}">

    <div class="slides"></div>
    <h3 class="title"></h3>
    <p class="description"></p>
    <a class="prev">
      <slot name="prev">‹</slot>
    </a>
    <a class="next">
      <slot name="next">›</slot>
    </a>
    <a v-if="!carousel" class="close">
      <slot name="close">X</slot>
    </a>
    <ol v-if="!carousel" class="indicator"></ol>
    <a v-if="carousel" class="play-pause"></a>
  </div>
</template>

<script>
  //use blueimp-gallery.css instead of blueimp-gallery.min.css,
  // because we do not need blueimp-gallery-indicator.css and blueimp-gallery-video.css
  import 'blueimp-gallery/css/blueimp-gallery.css';
  import blueimp from 'blueimp-gallery/js/blueimp-gallery.js';
  import 'blueimp-gallery/js/blueimp-gallery-fullscreen.js';
  // import 'blueimp-gallery/js/blueimp-gallery-video.js';
  // import 'blueimp-gallery/js/blueimp-gallery-youtube.js';

  export default {
    props: {
      images: {
        type: Array,
        default() {
          return [];
        },
      },

      options: {
        type: Object,
        default() {
          return {};
        },
      },

      carousel: {
        type: Boolean,
        default: false,
      },

      index: {
        type: Number,
      },

      id: {
        type: String,
        default: 'blueimp-gallery',
      },
    },

    data() {
      return {
        instance: null,
      };
    },

    watch: {
      index(value) {
        if (this.carousel) {
          return;
        }

        if (value !== null) {
          this.open(value);
        }
      },
    },

    mounted() {
      if (this.carousel) {
        this.open();
      }
    },

    destroyed() {
      this.close()
    },

    methods: {
      open(index = 0) {
        const instance = typeof blueimp.Gallery !== 'undefined' ? blueimp.Gallery : blueimp;

        const options = Object.assign({
          toggleControlsOnReturn: false,
          toggleControlsOnSlideClick: false,
          closeOnSlideClick: false,
          carousel: this.carousel,
          container: `#${this.id}`,
          index,
          onopen: () => this.$emit('onopen'),
          onopened: () => this.$emit('onopened'),
          onslide: this.onSlideCustom,
          onslideend: (index, slide) => this.$emit('onslideend', { index, slide }),
          onslidecomplete: (index, slide) => this.$emit('onslidecomplete', { index, slide }),
          onclose: () => this.$emit('close'),
          onclosed: () => this.$emit('onclosed'),
        }, this.options);

        if (this.carousel) {
          options.container = this.$el;
        }

        this.instance = instance(this.images, options);
      },
      close() {
        if (this.instance !== null) {
          this.instance.destroyEventListeners();
          //prevent onclose and onclosed event from being fired cyclically
          this.instance.options.onclose = null;
          this.instance.options.onclosed = null;
          this.instance.close(); //close and onclosed custom event will $emit here
          this.instance = null;
        }
      },
      onSlideCustom(index, slide) {
        this.$emit('onslide', { index, slide });

        const image = this.images[index];
        if (image !== undefined) {
          const text = image.description;
          const node = this.instance.container.find('.description');
          if (text) {
            node.empty();
            node[0].appendChild(document.createTextNode(text));
          }
        }
      },
    },
  };
</script>

<style>
  .blueimp-gallery > .description {
    position: absolute;
    top: 30px;
    left: 15px;
    color: #fff;
    display: none;
  }
  .blueimp-gallery-controls > .description {
    display: block;
  }
</style>
