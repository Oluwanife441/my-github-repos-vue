<script setup>
import PrevIcon from './icons/prevIcon.vue'
import NextIcon from './icons/nextIcon.vue'
import Skeleton from './Skeleton.vue'
import { ref, reactive, computed, onMounted } from 'vue'

const repoData = ref([])
const page = ref(1)
const view = ref('')
const currentPage = ref(1)
const loading = ref(false)
const perPage = ref(6)
const skeleton = reactive([...new Array(6)])
const searchQuery = ref('')

// Fetch Data
const fetchData = async () => {
  loading.value = true
  try {
    const res = await fetch(`https://api.github.com/users/Oluwanife441/repos`, {
      headers: {
        Accept: 'application/json'
      }
    })
    const data = await res.json()
    repoData.value = repoData.value.concat(data)
  } catch (error) {
    console.error(error)
  } finally {
    loading.value = false
  }
}

// Pagination
const prevPage = () => {
  if (currentPage.value === 1) return
  currentPage.value--
}

const nextPage = () => {
  currentPage.value++
}

// Computed Properties
const showMore = computed(() => {
  const filteredRepos = repoData.value.filter((repo) =>
    repo.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
  const start = (currentPage.value - 1) * perPage.value
  const end = start + perPage.value
  return filteredRepos.slice(start, end)
})

const lastPage = computed(() => {
  return Math.ceil(repoData.value.length / perPage.value)
})

// Styles
const inputStyle = {
  width: '50%',
  padding: '10px',
  margin: '10px',
  border: '1px solid #ccc',
  borderRadius: '5px'
}

// Lifecycle Hook
onMounted(fetchData)
</script>

<template>
  <div>
    <div style="display: flex; justify-content: center">
      <input type="text" placeholder="Search..." v-model="searchQuery" :style="inputStyle" />
    </div>
    <div class="repo-container">
      <Skeleton v-if="loading" v-for="n in skeleton" :key="n">{{ skeleton }}</Skeleton>
      <div v-else v-for="repo in showMore" class="repo-card" :key="repo.id">
        <router-link :to="`/details/${repo.name}`">
          <h2 class="repo-name">{{ repo.name }}</h2>
        </router-link>
        <p class="language">Language: {{ repo.language }}</p>
        <p class="date">Start date & time: {{ repo.created_at }}</p>
        <p class="visibility">Visibility: {{ repo.visibility }}</p>
      </div>
    </div>
    <div class="pagination">
      <button class="view-more" :class="{ disabled: currentPage === 1 }" @click="prevPage">
        <PrevIcon />
      </button>
      <p class="current-page">{{ currentPage }}</p>
      <button class="view-more" :class="{ disabled: currentPage === lastPage }" @click="nextPage">
        <NextIcon />
      </button>
    </div>
  </div>
</template>

<style scoped>
.repo-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  grid-gap: 30px;
  width: 90%;
  margin: 0 auto;
  margin-bottom: 5rem;
}

.repo-card,
.repodetail-card {
  border: 1px solid #ccc;
  padding: 13px;
  border-radius: 5px;
}

.repo-name {
  color: #ccc;
  font-size: 2rem;
  word-break: break-word;
}

p {
  padding: 5px 0;
}

.pagination {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin: 3rem 0;
}

button {
  background: none;
  border: none;
  cursor: pointer;
}

.view-more svg {
  width: 2rem;
}

.disabled {
  cursor: not-allowed;
  opacity: 0.5;
}
</style>
