<template>
    <main>
        <div class="container">
            <SearchBar @search="searchGithub" />
            <Panel :data="currentResult" />
        </div>
    </main>
</template>

<script setup>
import SearchBar from "./SearchBar.vue";
import Panel from "./Panel.vue";

import { ref } from 'vue';

const results = ref([])
const loading = ref(false)
const hasSearched = ref(false)

const currentResult = ref({});

const searchGithub = async (query) => {
  if (!query) return

  loading.value = true
  results.value = []
  hasSearched.value = true

  try {
    const res = await fetch(`https://api.github.com/search/users?q=${encodeURIComponent(query)}+type:user`)
    const data = await res.json()
    const basicUsers = data.items || []

    // Step 2: Fetch detailed info per user
    const detailedUsers = await Promise.all(
      basicUsers.slice(0, 5).map(async (user) => { // Limit to top 5 to avoid rate limits
        const detailRes = await fetch(`https://api.github.com/users/${user.login}`)
        return await detailRes.json()
      })
    )

    console.log("USERS", detailedUsers)

    currentResult.value = detailedUsers.length ? detailedUsers[0] : {};

  } catch (error) {
    console.error('GitHub API error:', error)
  } finally {
    loading.value = false
  }
}
</script>