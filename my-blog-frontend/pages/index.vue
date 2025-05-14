<template>
  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-bold text-gray-800 mb-6">Welcome to My Blog</h1>
    <div class="mb-6">
      <label for="search" class="block text-sm font-medium text-gray-700 mb-2">Search Posts</label>
      <input
        id="search"
        v-model="searchQuery"
        @keyup.enter="searchPosts"
        type="text"
        placeholder="Search by title or description..."
        class="p-2 border rounded-md w-full sm:w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
    </div>
    <div class="mb-6">
      <label for="category" class="block text-sm font-medium text-gray-700 mb-2">Filter by Category</label>
      <select
        id="category"
        v-model="selectedCategory"
        @change="fetchPosts"
        class="p-2 border rounded-md w-full sm:w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">All Categories</option>
        <option v-for="category in categories" :key="category.id" :value="category.id">
          {{ category.name || 'Unknown Category' }}
        </option>
      </select>
    </div>
    <div class="mb-4 text-gray-600">Posts: {{ posts.length }}</div>
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
  </div>
</template>

<script>
export default {
  data() {
    return {
      posts: [],
      categories: [],
      selectedCategory: '',
      searchQuery: '',
    };
  },
  methods: {
    async fetchPosts() {
      try {
        let url = `http://localhost:1337/api/articles?populate[author]=true&populate[categorys]=true`;
        if (this.selectedCategory) {
          url += `&filters[categorys][id][$eq]=${this.selectedCategory}`;
        }
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
    searchPosts() {
      if (this.searchQuery.trim()) {
        this.$router.push(`/search?query=${encodeURIComponent(this.searchQuery)}`);
      }
    },
    truncateDescription(text) {
      return text && text.length > 80 ? text.slice(0, 80) + '...' : text || '';
    },
  },
  async created() {
    try {

      const categoriesResponse = await fetch(`http://localhost:1337/api/categories`);
      if (!categoriesResponse.ok) {
        throw new Error(`HTTP error! Status: ${categoriesResponse.status}`);
      }
      const categoriesData = await categoriesResponse.json();
      this.categories = Array.isArray(categoriesData.data) ? categoriesData.data.map(item => ({
        id: item.id,
        name: item.attributes.Cat || 'Unknown',
      })) : [];

      await this.fetchPosts();
    } catch (error) {
      this.posts = [];
      this.categories = [];
    }
  },
};
</script>