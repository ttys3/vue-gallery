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
  // because we do not need blueimp-gallery-video.css
  import 'blueimp-gallery/css/blueimp-gallery.css';
  import 'blueimp-gallery/css/blueimp-gallery-indicator.css';
  import 'blueimp-gallery/js/blueimp-helper.js';
  import blueimp from 'blueimp-gallery/js/blueimp-gallery.js';
  import 'blueimp-gallery/js/blueimp-gallery-fullscreen.js';
  import 'blueimp-gallery/js/blueimp-gallery-indicator.js';
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
          onclose: () => this.$emit('onclose'),
          onclosed: () => this.$emit('onclosed'),
        }, this.options);

        if (this.carousel) {
          options.container = this.$el;
        }

        this.instance = instance(this.images, options);
      },
      close() {
        if (this.instance !== null) {
          //do not call this.instance.close() twice if called by blueimp-gallery already
          if (this.instance.container[0].style.display == 'none') {
            // console.log('vue-gallery: do not process twice')
            this.instance = null;
            return
          }
          //prevent onclose and onclosed event from being fired cyclically
          this.instance.options.onclose = null;
          this.instance.options.onclosed = null;
          // this.instance.close() can not recover document.body.style.overflow value
          this.instance.close();
          // if this.instance.container has not fired up webkitTransitionEnd event,
          // we need call handleClose() manually
          if (this.instance.container[0].style.display != 'none') {
            // console.log('vue-gallery: call handleClose() manually')
            this.instance.handleClose();
          }
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
      enterFullScreen() {
        if (!this.instance.getFullScreenElement()) {
          this.instance.requestFullScreen(this.instance.container[0])
        }
      },
      exitFullScreen() {
        if (this.instance.getFullScreenElement() === this.instance.container[0]) {
          this.instance.exitFullScreen()
        }
      },
      toggleFullScreen() {
        if (this.instance.getFullScreenElement() === this.instance.container[0]) {
          this.instance.exitFullScreen()
        } else {
          this.instance.requestFullScreen(this.instance.container[0])
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
