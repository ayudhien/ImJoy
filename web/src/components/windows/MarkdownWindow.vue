<template>
  <div>
    <md-card v-if="plugin_info">
      <md-card-media
        v-if="plugin_info.cover && typeof plugin_info.cover === 'string'"
        md-ratio="16:9"
      >
        <img :src="plugin_info.cover" alt="plugin-cover" />
      </md-card-media>
      <div class="carousel" v-else-if="plugin_info.cover">
        <!-- carousel locator -->
        <input
          class="carousel-locator"
          v-for="(c, k) in plugin_info.cover"
          :key="k"
          :id="'slide-' + k"
          type="radio"
          name="carousel-radio"
          hidden=""
        />
        <!-- carousel container -->
        <div class="carousel-container">
          <!-- carousel item -->
          <figure
            class="carousel-item"
            v-for="(c, k) in plugin_info.cover"
            :key="k"
          >
            <img class="img-responsive rounded" :src="c" alt="plugin cover" />
          </figure>
        </div>
        <!-- carousel navigation -->
        <div class="carousel-nav">
          <label
            class="nav-item text-hide c-hand"
            v-for="(c, k) in plugin_info.cover"
            :key="k"
            :for="'slide-' + k"
            >{{ k }}</label
          >
        </div>
      </div>
      <md-card-header>
        <md-toolbar md-elevation="0">
          <div>
            <h2>
              <plugin-icon :icon="plugin_info.icon"></plugin-icon>
              {{ plugin_info.name + " " + plugin_info.badges }}
            </h2>
          </div>
          <div class="md-toolbar-section-end">
            <p>version:{{ plugin_info.version }}</p>
          </div>
        </md-toolbar>
      </md-card-header>
      <md-card-content>
        <div
          style="padding-left: 10px; padding-right: 5px; overflow: auto"
          v-if="w.data && w.data.source && w.data.source.trim() != ''"
          v-html="sanitizedMarked(w.data.source)"
        ></div>
        <h4 v-else>
          {{ plugin_info && plugin_info.description }}
          <br />
          This plugin has no documentation!
        </h4>
      </md-card-content>
    </md-card>
  </div>
</template>

<script>
import marked from "marked";
import DOMPurify from "dompurify";

export default {
  name: "markdown-window",
  type: "imjoy/markdown",
  props: {
    w: {
      type: Object,
      default: function() {
        return null;
      },
    },
    loaders: {
      type: Object,
      default: function() {
        return null;
      },
    },
  },
  data() {
    return {
      plugin_info: null,
    };
  },
  created() {
    //open link in a new tab
    const renderer = new marked.Renderer();
    renderer.link = function(href, title, text) {
      var link = marked.Renderer.prototype.link.call(this, href, title, text);
      return link.replace("<a", "<a target='_blank' ");
    };
    marked.setOptions({
      renderer: renderer,
    });
    DOMPurify.addHook("afterSanitizeAttributes", function(node) {
      // set all elements owning target to target=_blank
      if ("target" in node) {
        node.setAttribute("target", "_blank");
        // prevent https://www.owasp.org/index.php/Reverse_Tabnabbing
        node.setAttribute("rel", "noopener noreferrer");
      }
    });
    this.sanitizedMarked = mk => {
      return DOMPurify.sanitize(marked(mk));
    };
  },
  mounted() {
    this.plugin_info = this.w && this.w.data && this.w.data.plugin_info;
    this.$emit("init");
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@media screen and (max-width: 600px) {
  .h2,
  h2 {
    font-size: 1rem;
  }
}
</style>
