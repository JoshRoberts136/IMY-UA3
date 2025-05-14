<template>
  <div class="container mx-auto p-6">
    <h1 class="text-3xl font-bold text-gray-800 mb-6">Welcome to My Blog</h1>
    <div class="mb-6">
      <label for="category" class="block text-sm font-medium text-gray-700 mb-2">Filter by Category</label>
      <select
        id="category"
        v-model="selectedCategory"
        @change="filterPosts"
        class="p-2 border rounded-md w-full sm:w-64 focus:outline-none focus:ring-2 focus:ring-blue-500"
      >
        <option value="">All Categories</option>
        <option v-for="category in categories" :key="category" :value="category">
          {{ category }}
        </option>
      </select>
    </div>
    <div v-if="posts.length === 0" class="text-gray-500 text-center">No posts found.</div>
    <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
      <div
        v-for="post in posts"
        :key="post.id"
        class="p-4 border rounded-lg shadow-sm hover:shadow-md transition-shadow"
      >
        <h2 class="text-xl font-semibold text-gray-800 mb-2">{{ post.attributes.title }}</h2>
        <p class="text-gray-600 text-sm mb-2">By {{ post.attributes.author }}</p>
        <p class="text-gray-700 mb-3">{{ post.attributes.snippet }}</p>
        <p class="text-sm text-gray-500">Category: {{ post.attributes.category }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      posts: [],
      selectedCategory: '',
      categories: [
        'Technology',
        'Lifestyle',
        'Travel',
        'Food & Cooking',
        'Health & Fitness',
        'Fashion',
        'Entertainment',
        'Gaming',
        'Education',
        'Business & Finance',
        'Science',
        'Environment',
        'Art & Design',
        'Photography',
        'Music',
        'Sports',
        'Books & Literature',
        'History',
        'DIY & Crafts',
        'Parenting',
      ],
    };
  },
  async fetch() {
    try {
      const response = await this.$http.$get(`${process.env.STRAPI_URL}/api/blog-posts`);
      this.posts = response.data;
    } catch (error) {
      console.error('Error fetching posts:', error);
    }
  },
  methods: {
    async filterPosts() {
      try {
        let url = `${process.env.STRAPI_URL}/api/blog-posts`;
        if (this.selectedCategory) {
          url += `?filters[category][$eq]=${encodeURIComponent(this.selectedCategory)}`;
        }
        const response = await this.$http.$get(url);
        this.posts = response.data;
      } catch (error) {
        console.error('Error filtering posts:', error);
      }
    },
  },
};
</script>