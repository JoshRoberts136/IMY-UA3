<template>
  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-bold text-gray-800 mb-4">{{ post?.attributes?.title || 'Post Not Found' }}</h1>
    <p class="text-gray-600 text-sm mb-2">
      By {{ post?.attributes?.author?.name || 'Unknown Author' }}
    </p>
    <p class="text-gray-700 mb-3">{{ post?.attributes?.description || 'No description available.' }}</p>
    <p class="text-sm text-gray-500">
      Category: {{ post?.attributes?.categorys?.Cat || 'Uncategorized' }}
    </p>
    <nuxt-link to="/" class="text-blue-500 hover:underline mt-4 inline-block">Back to Home</nuxt-link>
  </div>
</template>

<script>
import { reactive } from 'vue';

export default {
  setup() {
    const state = reactive({
      post: null,
    });

    return { state };
  },
  async created() {
    try {
      const id = this.$route.params.id;
      const response = await fetch(
        `http://localhost:1337/api/articles/${id}?populate[author]=true&populate[categorys]=true`
      );
      if (!response.ok) {
        throw new Error(`HTTP error! Status: ${response.status}, ${await response.text()}`);
      }
      const data = await response.json();
      console.log('Single post response:', JSON.stringify(data, null, 2));
      this.state.post = data.data || null;
    } catch (error) {
      console.error('Error fetching post:', error.message);
      this.state.post = null;
    }
  },
};
</script>