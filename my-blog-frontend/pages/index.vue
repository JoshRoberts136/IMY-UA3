<template>
  <div class="container mx-auto p-6">
    <!-- Header -->
    <h1 class="text-3xl font-bold text-gray-800 mb-6">Welcome to My Blog</h1>

    <!-- Category Dropdown -->
    <div class="mb-6">
      <label for="category" class="block text-sm font-medium text-gray-700 mb-2">Filter by Category</label>
      <select
        id="category"
        v-model="selectedCategory"
        @change="filterPosts"
        class="p-2 border rounded-md w-full sm:w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">All Categories</option>
        <option v-for="category in categories" :key="category.id" :value="category.id">
          {{ category.attributes.Cat }}
        </option>
      </select>
    </div>

    <!-- Blog Posts -->
    <div v-if="posts.length === 0" class="text-gray-500 text-center">No posts found.</div>
    <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
      <nuxt-link
        v-for="post in posts"
        :key="post.id"
        :to="`/post/${post.id}`"
        class="p-4 border rounded-lg shadow-sm hover:shadow-md transition-shadow"
      >
        <h2 class="text-xl font-semibold text-gray-800 mb-2">{{ post.attributes.title }}</h2>
        <p class="text-gray-600 text-sm mb-2">
          By {{ post.attributes.author?.data?.attributes?.name || 'Unknown Author' }}
        </p>
        <p class="text-gray-700 mb-3">{{ truncateSnippet(post.attributes.description) }}</p>
        <p class="text-sm text-gray-500">
          Category: {{ post.attributes.categorys?.data?.attributes?.Cat || 'Uncategorized' }}
        </p>
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
    };
  },
  methods: {
    async filterPosts() {
      try {
        let url = `${process.env.STRAPI_URL}/api/articles?populate[author]=true&populate[categorys]=true`;
        if (this.selectedCategory) {
          url += `&filters[categorys][id][$eq]=${this.selectedCategory}`;
        }
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}, ${await response.text()}`);
        }
        const data = await response.json();
        console.log('Filter posts response:', JSON.stringify(data, null, 2)); // Debug
        this.posts = Array.isArray(data.data) ? data.data : [];
      } catch (error) {
        console.error('Error filtering posts:', error.message);
        this.posts = [];
      }
    },
    truncateSnippet(text) {
      return text && text.length > 80 ? text.slice(0, 80) + '...' : text || '';
    },
  },
  async created() {
    try {
      // Fetch categories
      const categoriesResponse = await fetch(`${process.env.STRAPI_URL}/api/categories`);
      if (!categoriesResponse.ok) {
        throw new Error(`HTTP error! Status: ${categoriesResponse.status}, ${await categoriesResponse.text()}`);
      }
      const categoriesData = await categoriesResponse.json();
      console.log('Categories response:', JSON.stringify(categoriesData, null, 2)); // Debug
      this.categories = Array.isArray(categoriesData.data) ? categoriesData.data : [];

      // Fetch posts
      await this.filterPosts(); // Use filterPosts to initialize posts
    } catch (error) {
      console.error('Error fetching data:', error.message);
      this.posts = [];
      this.categories = [];
    }
  },
};
</script>