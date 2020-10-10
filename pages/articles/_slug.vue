<template>
  <section>
    <Article :blok="story.content" />
  </section>
</template>

<script>
import Article from "~/components/Article.vue";

export default {
  components: {
    Article,
  },
  data() {
    return {
      story: {
        content: {},
      },
    };
  },
  mounted() {
    this.$storybridge.on("input", (event) => {
      if (event.story.id === this.story.id) {
        this.story.content = event.story.content;
      }
    });

    this.$storybridge.on(["published", "change"], (event) => {
      this.$nuxt.$router.go({
        path: this.$nuxt.$router.currentRoute,
        force: true,
      });
    });
  },
  asyncData(context) {
    let version =
      context.query._storyblok || context.isDev ? "draft" : "published";

    return context.app.$storyapi
      .get(`cdn/stories/articles/${context.params.slug}`, {
        version: version,
      })
      .then((res) => {
        return res.data;
      })
      .catch((res) => {
        if (!res.response) {
          console.error(res);
          context.error({
            statusCode: 404,
            message: "Failed to receive content form api",
          });
        } else {
          console.error(res.response.data);
          context.error({
            statusCode: res.response.status,
            message: res.response.data,
          });
        }
      });
  },
};
</script>