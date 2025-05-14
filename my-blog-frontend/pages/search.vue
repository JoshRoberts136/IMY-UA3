<template>
  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-bold text-gray-800 mb-6">Search Results</h1>
    <p class="text-gray-600 mb-4">Showing results for: "{{ query }}"</p>

    <div v-if="posts.length === 0" class="text-gray-500 text-center">No posts found.</div>
    <div v-else class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
      <nuxt-link
        v-for="post in posts"
        :key="post.id"
        :to="`/post/${post.id}`"
        class="p-4 border rounded-lg shadow-sm hover:shadow-md transition-shadow"
      >
        <h2 class="text-xl font-semibold text-gray-800 mb-2">{{ post.title }}</h2>
        <p class="text-gray-600 text-sm mb-2">By {{ post.author?.name || 'Unknown Author' }}</p>
        <p class="text-gray-700 mb-3">{{ truncateDescription(post.description) }}</p>
        <p class="text-sm text-gray-500">Category: {{ post.category?.name || 'Uncategorized' }}</p>
      </nuxt-link>
    </div>
    <nuxt-link to="/" class="text-blue-500 hover:underline mt-4 inline-block">Back to Home</nuxt-link>
  </div>
</template>

<script>
export default {
  data() {
    return {
      posts: [],
      query: this.$route.query.query || '',
    };
  },
  methods: {
    async fetchPosts() {
      try {
        const url = `http://localhost:1337/api/articles?populate[author]=true&populate[categorys]=true&filters[$or][0][title][$containsi]=${encodeURIComponent(this.query)}&filters[$or][1][description][$containsi]=${encodeURIComponent(this.query)}`;
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        const data = await response.json();
        this.posts = Array.isArray(data.data) ? data.data.map(item => ({
          id: item.id,
          title: item.attributes.title,
          description: item.attributes.description,
          author: item.attributes.author || null,
          category: item.attributes.categorys || null,
        })) : [];
      } catch (error) {
        this.posts = [];
      }
    },
    truncateDescription(text) {
      return text && text.length > 80 ? text.slice(0, 80) + '...' : text || '';
    },
  },
  async created() {
    await this.fetchPosts();
  },
};
</script>