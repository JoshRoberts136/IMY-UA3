<template>
  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-bold text-gray-800 mb-4">{{ post?.title || 'Post Not Found' }}</h1>
    <p class="text-gray-600 text-sm mb-2">By {{ post?.author?.name || 'Unknown Author' }}</p>
    <p class="text-gray-700 mb-3">{{ post?.description || 'No description available.' }}</p>
    <p class="text-sm text-gray-500">Category: {{ post?.category?.name || 'Uncategorized' }}</p>
    <nuxt-link to="/" class="text-blue-500 hover:underline mt-4 inline-block">Back to Home</nuxt-link>
  </div>
</template>

<script>
export default {
  data() {
    return {
      post: null,
    };
  },
  async created() {
    try {
      const id = this.$route.params.id;
      const response = await fetch(
        `http://localhost:1337/api/articles/${id}?populate[author]=true&populate[categorys]=true`
      );
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}`);
      }
      const data = await response.json();
      this.post = data.data ? {
        id: data.data.id,
        title: data.data.attributes.title,
        description: data.data.attributes.description,
        author: data.data.attributes.author || null,
        category: data.data.attributes.categorys || null,
      } : null;
    } catch (error) {
      this.post = null;
    }
  },
};
</script>