<template>
  <div :class="[darkMode ? 'dark bg-gray-900' : 'bg-gray-100']">
    <!-- Header avec recherche et dark mode -->
    <header class="sticky top-0 z-10 backdrop-blur-lg bg-opacity-90 p-4 border-b dark:border-gray-700 bg-white dark:bg-gray-800 shadow-md">
      <div class="max-w-7xl mx-auto flex justify-between items-center">
        <h1 class="text-2xl font-bold dark:text-white transition duration-300 ease-in-out transform hover:scale-105">Lecteur Musical AI</h1>
        <div class="flex items-center gap-4">
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Rechercher..."
            class="px-4 py-2 rounded-lg bg-gray-200 dark:bg-gray-700 border dark:border-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition duration-200 ease-in-out"
          />
          <button
            @click="toggleDarkMode"
            class="p-2 rounded-full hover:bg-gray-300 dark:hover:bg-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition transform duration-200 ease-in-out"
          >
            <Icon
              :name="darkMode ? 'ph:sun-bold' : 'ph:moon-bold'"
              class="w-6 h-6"
            />
          </button>
        </div>
      </div>
    </header>

    <!-- Grille des musiques -->
    <main class="max-w-7xl mx-auto p-4 py-8">
      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
        <div
          v-for="track in filteredTracks"
          :key="track.id"
          class="cursor-pointer transform transition-all duration-300 hover:scale-105"
          @click="selectTrack(track)"
        >
          <div class="bg-white dark:bg-gray-800 rounded-lg overflow-hidden shadow-lg transition duration-300 ease-in-out hover:shadow-xl hover:bg-gray-100 dark:hover:bg-gray-700">
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
      class="fixed bottom-0 left-0 right-0 bg-white dark:bg-gray-800 border-t dark:border-gray-700 p-4 z-50 shadow-lg rounded-t-3xl"
    >
      <div class="max-w-7xl mx-auto flex items-center gap-4">
        <img
          :src="currentTrack.coverUrl"
          :alt="currentTrack.title"
          class="w-16 h-16 rounded-lg shadow-md"
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
            class="p-2 rounded-full hover:bg-gray-300 dark:hover:bg-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transform transition-all duration-200 ease-in-out"
            @click="previousTrack"
          >
            <Icon name="ph:skip-back-bold" class="w-6 h-6" />
          </button>
          <button
            class="p-2 rounded-full hover:bg-gray-300 dark:hover:bg-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transform transition-all duration-200 ease-in-out"
            @click="togglePlay"
          >
            <Icon
              :name="isPlaying ? 'ph:pause-bold' : 'ph:play-bold'"
              class="w-6 h-6"
            />
          </button>
          <button
            class="p-2 rounded-full hover:bg-gray-300 dark:hover:bg-gray-600 dark:text-white focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transform transition-all duration-200 ease-in-out"
            @click="nextTrack"
          >
            <Icon name="ph:skip-forward-bold" class="w-6 h-6" />
          </button>
        </div>
      </div>
      <!-- Barre de progression et temps -->
      <div class="max-w-7xl mx-auto mt-2">
        <div class="flex items-center justify-between text-sm text-gray-500 dark:text-gray-400">
          <!-- Temps écoulé -->
          <span>{{ formattedCurrentTime }}</span>
          <div
            class="bg-gray-300 dark:bg-gray-600 rounded-full h-2 flex-1 mx-2 cursor-pointer relative"
            @click="seekToPosition"
          >
            <div
              class="bg-blue-500 h-full rounded-full absolute top-0 left-0"
              :style="{ width: `${progress}%` }"
            ></div>
          </div>
          <!-- Temps total -->
          <span>{{ formattedDuration }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const darkMode = ref(false)
const searchQuery = ref('')
const currentTrack = ref(null)
const isPlaying = ref(false)
const progress = ref(0)
const duration = ref(0)
const currentTime = ref(0)

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
    artist: "Baguette",
    coverUrl: "/cover/DigitalDreams.jpeg",
    audioUrl: "/music/Digital Dreams.mp3"
  },
  {
    id: 3,
    title: "Pour Vanessa ♥",
    artist: "Moi ☺",
    coverUrl: "/cover/Vanessa.jpeg",
    audioUrl: "/music/Pour toi Vanessa !.mp3"
  }
]

const filteredTracks = computed(() => {
  if (!searchQuery.value) return musicData
  const query = searchQuery.value.toLowerCase()
  return musicData.filter(track => 
    track.title.toLowerCase().includes(query) ||
    track.artist.toLowerCase().includes(query)
  )
})

const audio = ref(null)

onMounted(() => {
  audio.value = new Audio()
  audio.value.addEventListener('timeupdate', updateProgress)
  audio.value.addEventListener('loadedmetadata', handleLoadedMetadata)
  audio.value.addEventListener('ended', () => {
    nextTrack()
  })
})

onUnmounted(() => {
  if (audio.value) {
    audio.value.removeEventListener('timeupdate', updateProgress)
    audio.value.removeEventListener('loadedmetadata', handleLoadedMetadata)
    audio.value.removeEventListener('ended', () => {})
  }
})

function handleLoadedMetadata() {
  if (audio.value && !isNaN(audio.value.duration)) {
    duration.value = audio.value.duration
    progress.value = (audio.value.currentTime / audio.value.duration) * 100
  }
}

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
  currentTime.value = audio.value.currentTime
  progress.value = (currentTime.value / duration.value) * 100
}

function seekToPosition(event) {
  if (!audio.value) return
  
  const rect = event.target.getBoundingClientRect()
  const clickX = event.clientX - rect.left
  const seekPosition = (clickX / rect.width) * duration.value
  
  if (audio.value) {
    audio.value.currentTime = seekPosition
  }
}

function previousTrack() {
  const currentIndex = musicData.indexOf(currentTrack.value)
  const previousTrackIndex = (currentIndex - 1 + musicData.length) % musicData.length
  selectTrack(musicData[previousTrackIndex])
}

function nextTrack() {
  const currentIndex = musicData.indexOf(currentTrack.value)
  const nextTrackIndex = (currentIndex + 1) % musicData.length
  selectTrack(musicData[nextTrackIndex])
}

const formattedCurrentTime = computed(() => formatTime(currentTime.value))
const formattedDuration = computed(() => formatTime(duration.value))

function formatTime(seconds) {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = Math.floor(seconds % 60)
  return `${minutes}:${remainingSeconds < 10 ? '0' : ''}${remainingSeconds}`
}
</script>

<style scoped>
/* Styles pour les boutons et animations */
button:focus {
  outline: none;
}
</style>
