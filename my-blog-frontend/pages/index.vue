<template>
  <div class="container mx-auto p-6">
    <!-- Header -->
    <h1 class="text-3xl font-bold text-gray-800 mb-6">Welcome to My Blog</h1>

    <!-- Category Dropdown -->
    <div class="mb-6">
      <label for="category" class="block text-sm font-medium text-gray-700 mb-2">Filter by Category</label>
      <select
        id="category"
        v-model="state.selectedCategory"
        @change="filterPosts"
        class="p-2 border rounded-md w-full sm:w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">All Categories</option>
        <option v-for="category in state.categories" :key="category.id" :value="category.id">
          {{ category.attributes.Cat }}
        </option>
      </select>
    </div>

    <!-- Blog Posts -->
    <div v-if="!state.posts.length" class="text-gray-500 text-center">No posts found.</div>
    <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
      <nuxt-link
        v-for="post in state.posts"
        :key="post.id"
        :to="`/post/${post.id}`"
        class="p-4 border rounded-lg shadow-sm hover:shadow-md transition-shadow"
      >
        <h2 class="text-xl font-semibold text-gray-800 mb-2">{{ post.attributes.title }}</h2>
        <p class="text-gray-600 text-sm mb-2">
          By {{ post.attributes.author?.name || 'Unknown Author' }}
        </p>
        <p class="text-gray-700 mb-3">{{ truncateSnippet(post.attributes.description) }}</p>
        <p class="text-sm text-gray-500">
          Category: {{ post.attributes.categorys?.Cat || 'Uncategorized' }}
        </p>
      </nuxt-link>
    </div>
  </div>
</template>

<script>
import { reactive } from 'vue';

export default {
  setup() {
    const state = reactive({
      posts: [],
      categories: [],
      selectedCategory: '',
    });

    const filterPosts = async () => {
      try {
        let url = `http://localhost:1337/api/articles?populate[author]=true&populate[categorys]=true`;
        if (state.selectedCategory) {
          url += `&filters[categorys][id][$eq]=${state.selectedCategory}`;
        }
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}, ${await response.text()}`);
        }
        const data = await response.json();
        console.log('Filter posts response:', JSON.stringify(data, null, 2));
        state.posts = Array.isArray(data.data) ? data.data : [];
        console.log('Posts after assignment:', JSON.stringify(state.posts, null, 2)); // Debug
      } catch (error) {
        console.error('Error filtering posts:', error.message);
        state.posts = [];
      }
    };

    const truncateSnippet = (text) => {
      return text && text.length > 80 ? text.slice(0, 80) + '...' : text || '';
    };

    return {
      state,
      filterPosts,
      truncateSnippet,
    };
  },
  async created() {
    try {
      // Fetch categories
      const categoriesResponse = await fetch(`http://localhost:1337/api/categories`);
      if (!categoriesResponse.ok) {
        throw new Error(`HTTP error! Status: ${categoriesResponse.status}, ${await categoriesResponse.text()}`);
      }
      const categoriesData = await categoriesResponse.json();
      console.log('Categories response:', JSON.stringify(categoriesData, null, 2));
      this.state.categories = Array.isArray(categoriesData.data) ? categoriesData.data : [];

      // Fetch posts
      await this.filterPosts();
    } catch (error) {
      console.error('Error fetching data:', error.message);
      this.state.posts = [];
      this.state.categories = [];
    }
  },
};
</script>