<template>
    <div v-if="!limitReached" class="search-bar">
        <svg height="24" width="25" xmlns="http://www.w3.org/2000/svg">
            <path
                d="M10.609 0c5.85 0 10.608 4.746 10.608 10.58 0 2.609-.952 5-2.527 6.847l5.112 5.087a.87.87 0 01-1.227 1.233l-5.118-5.093a10.58 10.58 0 01-6.848 2.505C4.759 21.16 0 16.413 0 10.58 0 4.747 4.76 0 10.609 0zm0 1.74c-4.891 0-8.87 3.965-8.87 8.84 0 4.874 3.979 8.84 8.87 8.84a8.855 8.855 0 006.213-2.537l.04-.047a.881.881 0 01.058-.053 8.786 8.786 0 002.558-6.203c0-4.875-3.979-8.84-8.87-8.84z"
                fill="#0079ff" />
        </svg>
        <form action="" @submit.prevent="searchGithub">
            <input v-model="query" type="text" placeholder="Search GitHub username…" />
            <button type="submit">Search</button>
        </form>
    </div>

    <div v-if="limitReached" style="margin-top: 24px;">403 - API rate limit exceeded! <br>Try again after one hour.</div>
    <div v-else-if="hasSearched && !hasResults">No profiles match your query.</div>
    <div v-else-if="hasSearched && hasResults" style="margin-bottom: 24px;">{{ formatBigNumbers(totalMatches) }} profiles match your query.</div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
    hasSearched: {
        type: Boolean,
        default: false
    },
    hasResults: {
        type: Boolean,
        default: false
    },
    limitReached: {
        type: Boolean,
        default: false
    },
    totalMatches: {
        type: Number,
        default: 0
    }
})

const query = ref('');
const emit = defineEmits(["search"]);

const searchGithub = () => {
    emit("search", query.value);
}

const formatBigNumbers = (value) => {
    return (new Intl.NumberFormat("en-EN", { notation: "compact"})).format(value);
}
</script>