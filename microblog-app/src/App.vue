<script setup>
import { ref, onMounted, watch, computed } from 'vue'


import avatar1 from './assets/avatars/avatar1.jpg'
import avatar2 from './assets/avatars/avatar2.jpg'
import avatar3 from './assets/avatars/avatar3.jpg'
import avatar4 from './assets/avatars/avatar4.jpg'
import avatar5 from './assets/avatars/avatar5.jpg'
import avatar6 from './assets/avatars/avatar6.jpg'
import avatar7 from './assets/avatars/avatar7.jpg'

const avatars = [
  avatar1,
  avatar2,
  avatar3,
  avatar4,
  avatar5,
  avatar6,
  avatar7,
]


const postTypes = {
  Games: 'primary',
  Sports: 'secondary',
  Study: 'accent',
  Music: 'info',
  Art: 'success'
}

const currentPage = ref('feed')
const theme = ref('light')
const authors = ref([])
const posts = ref([])
const selectedAuthorId = ref(null)
const filterType = ref('all')


const newAuthorName = ref('')
const selectedAvatar = ref(avatars[0])
const newPostAuthor = ref('')
const newPostTitle = ref('')
const newPostDescription = ref('')
const newPostType = ref('Games')


onMounted(() => {
  const savedTheme = localStorage.getItem('microblog-theme')
  if (savedTheme) {
    theme.value = savedTheme
  }

  const savedAuthors = localStorage.getItem('microblog-authors')
  if (savedAuthors) {
    authors.value = JSON.parse(savedAuthors)
  }

  const savedPosts = localStorage.getItem('microblog-posts')
  if (savedPosts) {
    posts.value = JSON.parse(savedPosts)
  }
})


watch(theme, (newTheme) => {
  localStorage.setItem('microblog-theme', newTheme)
  document.documentElement.setAttribute('data-theme', newTheme)
})


watch(authors, (newAuthors) => {
  localStorage.setItem('microblog-authors', JSON.stringify(newAuthors))
}, { deep: true })


watch(posts, (newPosts) => {
  localStorage.setItem('microblog-posts', JSON.stringify(newPosts))
}, { deep: true })

const createAuthor = () => {
  if (!newAuthorName.value.trim()) return

  const newAuthor = {
    id: Date.now(),
    name: newAuthorName.value,
    avatar: selectedAvatar.value
  }

  authors.value.push(newAuthor)
  newAuthorName.value = ''
  selectedAvatar.value = avatars[0]
  selectedAuthorId.value = newAuthor.id
  currentPage.value = 'profile'
}

const createPost = () => {
  if (!newPostAuthor.value || !newPostTitle.value.trim() || !newPostDescription.value.trim()) return

  const newPost = {
    id: Date.now(),
    authorId: parseInt(newPostAuthor.value),
    title: newPostTitle.value,
    description: newPostDescription.value,
    type: newPostType.value,
    date: new Date().toLocaleDateString('uk-UA')
  }

  posts.value.unshift(newPost)
  newPostAuthor.value = ''
  newPostTitle.value = ''
  newPostDescription.value = ''
  newPostType.value = 'Games'
  currentPage.value = 'feed'
}

const getAuthorById = (id) => {
  return authors.value.find(a => a.id === id)
}

const getAuthorPosts = (authorId) => {
  return posts.value.filter(p => p.authorId === authorId)
}

const getPostsByType = (authorId, type) => {
  return posts.value.filter(p => p.authorId === authorId && p.type === type).length
}

const goToProfile = (authorId) => {
  selectedAuthorId.value = authorId
  currentPage.value = 'profile'
  filterType.value = 'all'

  setTimeout(() => {
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }, 500)
}

const filteredPosts = computed(() => {
  if (currentPage.value === 'profile' && selectedAuthorId.value) {
    const authorPosts = getAuthorPosts(selectedAuthorId.value)
    return filterType.value === 'all'
        ? authorPosts
        : authorPosts.filter(p => p.type === filterType.value)
  }
  return posts.value
})
</script>

<template>
  <div class="min-h-screen flex flex-col" :data-theme="theme">
    <!-- Header -->
    <header class="navbar bg-base-200 shadow-lg">
      <div class="flex-1">
        <a class="btn btn-ghost text-xl">Мікроблог</a>
      </div>
      <div class="flex-none gap-2">
        <button
            class="btn btn-ghost btn-sm"
            @click="currentPage = 'feed'"
        >
           Стрічка
        </button>
        <button
            class="btn btn-ghost btn-sm"
            @click="currentPage = 'createAuthor'"
        >
           Автор
        </button>
        <button
            class="btn btn-ghost btn-sm"
            @click="currentPage = 'createPost'"
        >
           Пост
        </button>
        <select
            class="select select-bordered select-sm"
            v-model="theme"
        >
          <option value="light">Світла</option>
          <option value="dark">Темна</option>
        </select>
      </div>
    </header>


    <main class="flex-1 container mx-auto p-4">

      <div v-if="currentPage === 'feed'">
        <h1 class="text-3xl font-bold mb-6">Стрічка новин</h1>
        <div v-if="posts.length === 0" class="alert">
          <span>Поки немає постів. Створіть перший!</span>
        </div>
        <div v-else class="space-y-4">
          <div
              v-for="post in posts"
              :key="post.id"
              class="card bg-base-100 shadow-xl transition-all hover:shadow-2xl"
          >
            <div class="card-body">
              <div class="flex items-center gap-3 mb-2">
                <div
                    class="avatar cursor-pointer"
                    @click="goToProfile(post.authorId)"
                >
                  <div class="w-12 rounded-full">
                    <img :src="getAuthorById(post.authorId)?.avatar" />
                  </div>
                </div>
                <div class="flex-1">
                  <h3
                      class="font-bold cursor-pointer hover:underline"
                      @click="goToProfile(post.authorId)"
                  >
                    {{ getAuthorById(post.authorId)?.name }}
                  </h3>
                  <p class="text-sm opacity-60">{{ post.date }}</p>
                </div>
                <div class="badge badge-lg" :class="{
                  'badge-primary': post.type === 'Games',
                  'badge-secondary': post.type === 'Sports',
                  'badge-accent': post.type === 'Study',
                  'badge-info': post.type === 'Music',
                  'badge-success': post.type === 'Art'
                }">
                  {{ post.type }}
                </div>
              </div>
              <h2 class="card-title">{{ post.title }}</h2>
              <p class="opacity-80">{{ post.description }}</p>
            </div>
          </div>
        </div>
      </div>


      <div v-if="currentPage === 'createAuthor'" class="max-w-2xl mx-auto">
        <h1 class="text-3xl font-bold mb-6">Створити автора</h1>
        <div class="card bg-base-100 shadow-xl">
          <div class="card-body">
            <div class="form-control">
              <label class="label">
                <span class="label-text">Ім'я автора</span>
              </label>
              <input
                  type="text"
                  placeholder="Введіть ім'я"
                  class="input input-bordered"
                  v-model="newAuthorName"
              />
            </div>

            <div class="form-control mt-4">
              <label class="label">
                <span class="label-text">Оберіть аватар</span>
              </label>
              <div class="grid grid-cols-4 gap-4">
                <div
                    v-for="(avatar, index) in avatars"
                    :key="index"
                    class="cursor-pointer p-2 rounded-lg border-2 transition-all hover:scale-105"
                    :class="selectedAvatar === avatar ? 'border-primary' : 'border-base-300'"
                    @click="selectedAvatar = avatar"
                >
                  <img :src="avatar" class="w-full h-24 object-cover rounded-lg" />
                </div>
              </div>
            </div>

            <div class="card-actions justify-end mt-6">
              <button
                  class="btn btn-primary"
                  @click="createAuthor"
              >
                Створити автора
              </button>
            </div>
          </div>
        </div>
      </div>


      <div v-if="currentPage === 'createPost'" class="max-w-2xl mx-auto">
        <h1 class="text-3xl font-bold mb-6">Створити пост</h1>
        <div v-if="authors.length === 0" class="alert alert-warning">
          <span>Спочатку створіть автора!</span>
        </div>
        <div v-else class="card bg-base-100 shadow-xl">
          <div class="card-body">
            <div class="form-control">
              <label class="label">
                <span class="label-text">Автор</span>
              </label>
              <select
                  class="select select-bordered"
                  v-model="newPostAuthor"
              >
                <option value="">Оберіть автора</option>
                <option
                    v-for="author in authors"
                    :key="author.id"
                    :value="author.id"
                >
                  {{ author.name }}
                </option>
              </select>
            </div>

            <div class="form-control mt-4">
              <label class="label">
                <span class="label-text">Назва посту</span>
              </label>
              <input
                  type="text"
                  placeholder="Введіть назву"
                  class="input input-bordered"
                  v-model="newPostTitle"
              />
            </div>

            <div class="form-control mt-4">
              <label class="label">
                <span class="label-text">Опис посту</span>
              </label>
              <textarea
                  class="textarea textarea-bordered h-24"
                  placeholder="Введіть опис"
                  v-model="newPostDescription"
              ></textarea>
            </div>

            <div class="form-control mt-4">
              <label class="label">
                <span class="label-text">Тип посту</span>
              </label>
              <select
                  class="select select-bordered"
                  v-model="newPostType"
              >
                <option
                    v-for="(color, type) in postTypes"
                    :key="type"
                    :value="type"
                >
                  {{ type }}
                </option>
              </select>
            </div>

            <div class="card-actions justify-end mt-6">
              <button
                  class="btn btn-primary"
                  @click="createPost"
              >
                Створити пост
              </button>
            </div>
          </div>
        </div>
      </div>


      <div v-if="currentPage === 'profile' && selectedAuthorId">
        <div class="card bg-base-100 shadow-xl mb-6">
          <div class="card-body">
            <div class="flex items-center gap-6">
              <div class="avatar">
                <div class="w-24 rounded-full">
                  <img :src="getAuthorById(selectedAuthorId)?.avatar" />
                </div>
              </div>
              <div class="flex-1">
                <h1 class="text-3xl font-bold mb-2">{{ getAuthorById(selectedAuthorId)?.name }}</h1>
                <div class="stats shadow">
                  <div class="stat">
                    <div class="stat-title">Всього постів</div>
                    <div class="stat-value">
                      <span class="countdown">
                        <span :style="{'--value': getAuthorPosts(selectedAuthorId).length}"></span>
                      </span>
                    </div>
                  </div>
                  <template v-for="(color, type) in postTypes" :key="type">
                    <div v-if="getPostsByType(selectedAuthorId, type) > 0" class="stat">
                      <div class="stat-title">{{ type }}</div>
                      <div class="stat-value text-primary">
                        <span class="countdown">
                          <span :style="{'--value': getPostsByType(selectedAuthorId, type)}"></span>
                        </span>
                      </div>
                    </div>
                  </template>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="flex justify-between items-center mb-4">
          <h2 class="text-2xl font-bold">Пости автора</h2>
          <select
              class="select select-bordered"
              v-model="filterType"
          >
            <option value="all">Всі типи</option>
            <option
                v-for="(color, type) in postTypes"
                :key="type"
                :value="type"
            >
              {{ type }}
            </option>
          </select>
        </div>

        <div v-if="filteredPosts.length === 0" class="alert">
          <span>Немає постів для відображення</span>
        </div>
        <div v-else class="space-y-4">
          <div
              v-for="post in filteredPosts"
              :key="post.id"
              class="card bg-base-100 shadow-xl transition-all hover:shadow-2xl"
          >
            <div class="card-body">
              <div class="flex justify-between items-start mb-2">
                <p class="text-sm opacity-60">{{ post.date }}</p>
                <div class="badge badge-lg" :class="{
                  'badge-primary': post.type === 'Games',
                  'badge-secondary': post.type === 'Sports',
                  'badge-accent': post.type === 'Study',
                  'badge-info': post.type === 'Music',
                  'badge-success': post.type === 'Art'
                }">
                  {{ post.type }}
                </div>
              </div>
              <h2 class="card-title">{{ post.title }}</h2>
              <p class="opacity-80">{{ post.description }}</p>
            </div>
          </div>
        </div>
      </div>
    </main>


  </div>
</template>