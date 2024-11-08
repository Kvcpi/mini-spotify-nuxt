// pages/index.vue
<template>
  <div :class="[darkMode ? 'dark bg-gray-900' : 'bg-gray-50']">
    <!-- Header avec recherche et dark mode -->
    <header class="sticky top-0 z-10 backdrop-blur-lg bg-opacity-90 p-4 border-b dark:border-gray-700">
      <div class="max-w-7xl mx-auto flex justify-between items-center">
        <h1 class="text-2xl font-bold dark:text-white">Lecteur Musical AI</h1>
        <div class="flex items-center gap-4">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Rechercher..."
            class="px-4 py-2 rounded-lg bg-white dark:bg-gray-800 border dark:border-gray-700 dark:text-white"
          />
          <button
            @click="toggleDarkMode"
            class="p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700"
          >
            <Icon
              :name="darkMode ? 'ph:sun-bold' : 'ph:moon-bold'"
              class="w-6 h-6 dark:text-white"
            />
          </button>
        </div>
      </div>
    </header>

    <!-- Grille des musiques -->
    <main class="max-w-7xl mx-auto p-4">
      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        <div
          v-for="track in filteredTracks"
          :key="track.id"
          class="cursor-pointer transform transition-all duration-300 hover:scale-105"
          @click="selectTrack(track)"
        >
          <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-lg">
            <img
              :src="track.coverUrl"
              :alt="track.title"
              class="w-full aspect-square object-cover"
            />
            <div class="p-4">
              <h3 class="font-semibold text-gray-900 dark:text-white truncate">
                {{ track.title }}
              </h3>
              <p class="text-sm text-gray-500 dark:text-gray-400 truncate">
                {{ track.artist }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Lecteur audio -->
    <div
      v-if="currentTrack"
      class="fixed bottom-0 left-0 right-0 bg-white dark:bg-gray-800 border-t dark:border-gray-700 p-4"
    >
      <div class="max-w-7xl mx-auto flex items-center gap-4">
        <img
          :src="currentTrack.coverUrl"
          :alt="currentTrack.title"
          class="w-16 h-16 rounded-lg"
        />
        <div class="flex-1">
          <h3 class="font-semibold dark:text-white">{{ currentTrack.title }}</h3>
          <p class="text-sm text-gray-500 dark:text-gray-400">
            {{ currentTrack.artist }}
          </p>
        </div>
        <!-- Contrôles de lecture -->
        <div class="flex items-center gap-4">
          <button
            class="p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700 dark:text-white"
            @click="previousTrack"
          >
            <Icon name="ph:skip-back-bold" class="w-6 h-6" />
          </button>
          <button
            class="p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700 dark:text-white"
            @click="togglePlay"
          >
            <Icon
              :name="isPlaying ? 'ph:pause-bold' : 'ph:play-bold'"
              class="w-6 h-6"
            />
          </button>
          <button
            class="p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700 dark:text-white"
            @click="nextTrack"
          >
            <Icon name="ph:skip-forward-bold" class="w-6 h-6" />
          </button>
        </div>
      </div>
      <!-- Barre de progression -->
      <div class="max-w-7xl mx-auto mt-2">
        <div class="bg-gray-200 dark:bg-gray-700 rounded-full h-1">
          <div
            class="bg-blue-500 h-full rounded-full"
            :style="{ width: `${progress}%` }"
          ></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const darkMode = ref(false)
const searchQuery = ref('')
const currentTrack = ref(null)
const isPlaying = ref(false)
const progress = ref(0)

// Exemple de données (à remplacer par vos musiques générées)
const musicData = [
  {
    id: 1,
    title: "Dancing shadows",
    artist: "KvcpiZebi",
    coverUrl: "/cover/dancingShadows.jpeg",
    audioUrl: "/music/Dancing Shadows.mp3"
  },
  {
    id: 2,
    title: "Digital Dreams",
    artist: "MohamedMoussa",
    coverUrl: "/cover/DigitalDreams.jpeg",
    audioUrl: "/music/Digital Dreams.mp3"
  },
  {
    id: 3,
    title: "Sunset Melody",
    artist: "Virtual Band",
    coverUrl: "/placeholder-cover-3.jpg",
    audioUrl: "/music/track3.mp3"
  }
]

// Filtrage des musiques
const filteredTracks = computed(() => {
  if (!searchQuery.value) return musicData
  const query = searchQuery.value.toLowerCase()
  return musicData.filter(track => 
    track.title.toLowerCase().includes(query) ||
    track.artist.toLowerCase().includes(query)
  )
})

// Gestion de l'audio
const audio = ref(null)

onMounted(() => {
  audio.value = new Audio()
  audio.value.addEventListener('timeupdate', updateProgress)
  audio.value.addEventListener('ended', () => {
    nextTrack()
  })
})

onUnmounted(() => {
  if (audio.value) {
    audio.value.removeEventListener('timeupdate', updateProgress)
    audio.value.removeEventListener('ended', () => {})
  }
})

// Fonctions de contrôle
function toggleDarkMode() {
  darkMode.value = !darkMode.value
}

function selectTrack(track) {
  currentTrack.value = track
  if (audio.value) {
    audio.value.src = track.audioUrl
    audio.value.play()
    isPlaying.value = true
  }
}

function togglePlay() {
  if (!audio.value) return
  
  if (isPlaying.value) {
    audio.value.pause()
  } else {
    audio.value.play()
  }
  isPlaying.value = !isPlaying.value
}

function updateProgress() {
  if (!audio.value) return
  const value = (audio.value.currentTime / audio.value.duration) * 100
  progress.value = value || 0
}

function nextTrack() {
  const currentIndex = musicData.findIndex(track => track.id === currentTrack.value?.id)
  const nextIndex = (currentIndex + 1) % musicData.length
  selectTrack(musicData[nextIndex])
}

function previousTrack() {
  const currentIndex = musicData.findIndex(track => track.id === currentTrack.value?.id)
  const prevIndex = (currentIndex - 1 + musicData.length) % musicData.length
  selectTrack(musicData[prevIndex])
}
</script>