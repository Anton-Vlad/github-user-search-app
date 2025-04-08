<template>
    <main>
        <div class="container">
            <SearchBar 
                :hasSearched="hasSearched" 
                :hasResults="hasResults" 
                :totalMatches="totalMatches"
                :limitReached="rateLimitReached"
                @search="searchGithub" 
            />
            <Panel v-if="hasResults" :data="currentResult" />
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
const hasResults = ref(false)
const currentResult = ref({})
const totalMatches = ref(0)
const rateLimitReached = ref(false)

const searchGithub = async (query) => {
  if (!query) return

  loading.value = true
  results.value = []
  hasSearched.value = true

  try {
    const res = await fetch(`https://api.github.com/search/users?q=${encodeURIComponent(query)}+type:user`)
    if (!res.ok) throw new Error(`Search failed: ${res.status} ${res.statusText}`)
    const data = await res.json()
    const basicUsers = data.items || []
    totalMatches.value = data.total_count;

    if (!totalMatches.value) {
        currentResult.value = {};
        hasResults.value = false;
    }

    // Step 2: Fetch detailed info per user
    const detailedUsers = await Promise.all(
      basicUsers.slice(0, 5).map(async (user) => { // Limit to top 5 to avoid rate limits
        const detailRes = await fetch(`https://api.github.com/users/${user.login}`)
        if (!detailRes.ok) {
            if (detailRes.status === '403') {

            } 
            throw new Error(`User fetch faileddd: ${detailRes.status} ${detailRes.statusText}`)
        }
        return await detailRes.json()
      })
    )

    currentResult.value = detailedUsers.length ? detailedUsers[0] : {};
    hasResults.value = detailedUsers.length > 0;

  } catch (error) {
    // console.log('GitHub API error:', error.message)
    rateLimitReached.value = true;
    currentResult.value = {};
    hasResults.value = false;
    
  } finally {
    loading.value = false
  }
}
</script>