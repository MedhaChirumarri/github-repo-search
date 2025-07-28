<template>
  <v-app>
    <v-container>
      <v-text-field
        v-model="query"
        label="Search GitHub Repos"
        @keyup.enter="searchRepos"
        append-inner-icon="mdi-magnify"
        clearable
      ></v-text-field>

      <v-list v-if="results.length > 0">
        <v-list-item
          v-for="repo in results"
          :key="repo.id"
          :href="repo.html_url"
          target="_blank"
        >
          <v-list-item-title>{{ repo.full_name }}</v-list-item-title>
          <v-list-item-subtitle>{{ repo.description }}</v-list-item-subtitle>
        </v-list-item>
      </v-list>

      <v-pagination
        v-if="totalPages > 1"
        v-model="page"
        :length="totalPages"
      ></v-pagination>

    </v-container>
  </v-app>
</template>

<script lang="ts" setup>
import { ref, watch } from 'vue'
import axios from 'axios'

const query = ref('')
const results = ref<any[]>([])
const page = ref(1)
const totalPages = ref(0)
const perPage = 10

// Fetch data when the query changes
watch(query, () => {
  page.value = 1
  fetchRepos()
})

// Fetch data when the page changes
watch(page, (newPage, oldPage) => {
  if (query.value && newPage !== oldPage) {
    fetchRepos()
  }
})

const searchRepos = () => {
  if (query.value) {
    page.value = 1
    fetchRepos()
  }
}

const fetchRepos = async () => {
  if (!query.value) return

  try {
    const { data } = await axios.get('https://api.github.com/search/repositories', {
      params: {
        q: query.value,
        page: page.value,
        per_page: perPage,
      }
    })

    results.value = data.items
    totalPages.value = Math.ceil(Math.min(data.total_count, 1000) / perPage) // GitHub search caps at 1000
  } catch (error) {
    console.error('Error fetching repos:', error)
    results.value = []
    totalPages.value = 0
  }
}
</script>
