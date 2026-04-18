<template>
  <div id="app" @click.self="langOpen = false">
    <!-- Top-left: region selector -->
    <div class="top-controls top-controls--left">
      <div class="lang-dropdown">
        <button class="btn-topbar" @click="regionOpen = !regionOpen">
          {{ t.regions[selectedRegion] }}
        </button>
        <div v-if="regionOpen" class="lang-menu lang-menu--left">
          <button
            v-for="r in regions"
            :key="r.code"
            class="lang-option"
            :class="{ active: selectedRegion === r.code }"
            @click="setRegion(r.code)"
          >
            {{ t.regions[r.code] }}
          </button>
        </div>
      </div>
    </div>
    <!-- Top-right controls -->
    <div class="top-controls">
      <!-- Language dropdown -->
      <div class="lang-dropdown" v-click-outside="() => langOpen = false">
        <button class="btn-topbar" @click="langOpen = !langOpen">
          <img :src="currentLang.flagImg" class="lang-flag-img" :alt="currentLang.label" />
          {{ currentLang.label }}
        </button>
        <div v-if="langOpen" class="lang-menu">
          <button
            v-for="lang in languages"
            :key="lang.code"
            class="lang-option"
            :class="{ active: locale === lang.code }"
            @click="setLang(lang.code)"
          >
            <img :src="lang.flagImg" class="lang-flag-img" :alt="lang.label" />
            {{ lang.label }}
          </button>
        </div>
      </div>

      <!-- Fullscreen toggle -->
      <button class="btn-topbar" @click="toggleFullscreen">
        <span>{{ isFullscreen ? '✕ ' + t.exitFullscreen : '⛶ ' + t.fullscreen }}</span>
      </button>
    </div>

    <!-- Step: start -->
    <div v-if="step === 'start'" class="screen">
      <h1>🌍 {{ t.appTitle }}</h1>
      <p class="subtitle">{{ t.appSubtitle }}</p>
      <button class="btn btn-primary" @click="step = 'ready'">{{ t.startGame }}</button>
    </div>

    <!-- Step: ready -->
    <div v-else-if="step === 'ready'" class="screen">
      <h2>{{ t.readyTitle }}</h2>
      <p class="subtitle">{{ t.readySubtitle }}</p>
      <button class="btn btn-primary" @click="startRound">{{ t.letsGo }}</button>
      <button class="btn btn-secondary" @click="step = 'start'">{{ t.back }}</button>
    </div>

    <!-- Step: flag -->
    <div v-else-if="step === 'flag'" class="screen">
      <h2>{{ t.flagQuestion }}</h2>
      <div class="flag-container">
        <img
          :src="flagUrl"
          :alt="`Flag of ${currentCountry.name}`"
          class="flag-img"
        />
      </div>
      <button class="btn btn-primary" @click="step = 'answer'">{{ t.revealAnswer }}</button>
      <button class="btn btn-secondary" @click="step = 'ready'">{{ t.skip }}</button>
    </div>

    <!-- Step: answer -->
    <div v-else-if="step === 'answer'" class="screen">
      <h2>{{ t.answerTitle }}</h2>
      <div class="flag-container">
        <img
          :src="flagUrl"
          :alt="`Flag of ${currentCountry.name}`"
          class="flag-img flag-img--dimmed"
        />
        <div class="answer-overlay">
          <div class="answer-badge">🏳️ {{ countryName }}</div>
        </div>
      </div>
      <div class="action-row">
        <button class="btn btn-primary" @click="startRound">{{ t.nextFlag }}</button>
        <button class="btn btn-secondary" @click="step = 'start'">{{ t.backToStart }}</button>
      </div>
    </div>
  </div>
</template>

<script>
import { countries } from './data/countries.js'

const i18n = {
  pl: {
    appTitle: 'Zgadnij Flagę',
    appSubtitle: 'Jak dobrze znasz flagi świata?',
    startGame: 'Rozpocznij grę',
    readyTitle: 'Jesteś gotowy?',
    readySubtitle: 'Pojawi się losowa flaga — spróbuj odgadnąć kraj!',
    letsGo: 'Tak, zaczynamy! 🚀',
    back: 'Wstecz',
    flagQuestion: 'Do jakiego kraju należy ta flaga?',
    revealAnswer: 'Odkryj odpowiedź 👁️',
    skip: 'Pomiń',
    answerTitle: 'Odpowiedź to…',
    nextFlag: 'Następna flaga ➡️',
    backToStart: 'Powrót do początku',
    fullscreen: 'Pełny ekran',
    exitFullscreen: 'Wyjdź',
    regions: {
      world: 'Świat',
      europe: 'Europa',
      asia: 'Azja',
      africa: 'Afryka',
      americas: 'Ameryki',
      oceania: 'Oceania',
    },
  },
  en: {
    appTitle: 'Flag Guesser',
    appSubtitle: 'How well do you know the flags of the world?',
    startGame: 'Start Game',
    readyTitle: 'Are you ready?',
    readySubtitle: 'A random flag will appear — try to guess the country!',
    letsGo: "Yes, let's go! 🚀",
    back: 'Back',
    flagQuestion: 'Which country does this flag belong to?',
    revealAnswer: 'Reveal Answer 👁️',
    skip: 'Skip',
    answerTitle: 'The answer is…',
    nextFlag: 'Next Flag ➡️',
    backToStart: 'Back to Start',
    fullscreen: 'Fullscreen',
    exitFullscreen: 'Exit',
    regions: {
      world: 'World',
      europe: 'Europe',
      asia: 'Asia',
      africa: 'Africa',
      americas: 'Americas',
      oceania: 'Oceania',
    },
  },
}

export default {
  name: 'App',
  data() {
    return {
      step: 'start',
      currentCountry: null,
      isFullscreen: false,
      locale: 'pl',
      langOpen: false,
      regionOpen: false,
      selectedRegion: 'world',
      regions: [
        { code: 'world',    icon: '🌍' },
        { code: 'europe',   icon: '🇪🇺' },
        { code: 'asia',     icon: '🌏' },
        { code: 'africa',   icon: '🌍' },
        { code: 'americas', icon: '🌎' },
        { code: 'oceania',  icon: '🌊' },
      ],
      languages: [
        { code: 'pl', flagImg: 'https://flagcdn.com/w40/pl.png', label: 'Polski' },
        { code: 'en', flagImg: 'https://flagcdn.com/w40/gb.png', label: 'English' },
      ],
    }
  },
  computed: {
    t() {
      return i18n[this.locale]
    },
    currentLang() {
      return this.languages.find(l => l.code === this.locale)
    },
    currentRegion() {
      return this.regions.find(r => r.code === this.selectedRegion)
    },
    filteredCountries() {
      if (this.selectedRegion === 'world') return countries
      return countries.filter(c => c.region === this.selectedRegion)
    },
    flagUrl() {
      if (!this.currentCountry) return ''
      return `https://flagcdn.com/w2560/${this.currentCountry.code}.png`
    },
    countryName() {
      if (!this.currentCountry) return ''
      return this.locale === 'pl' ? this.currentCountry.namePl : this.currentCountry.name
    },
  },
  methods: {
    startRound() {
      const pool = this.filteredCountries
      const index = Math.floor(Math.random() * pool.length)
      this.currentCountry = pool[index]
      this.step = 'flag'
    },
    toggleFullscreen() {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen()
      } else {
        document.exitFullscreen()
      }
    },
    setLang(code) {
      this.locale = code
      this.langOpen = false
    },
    setRegion(code) {
      this.selectedRegion = code
      this.regionOpen = false
    },
  },
  mounted() {
    document.addEventListener('fullscreenchange', () => {
      this.isFullscreen = !!document.fullscreenElement
    })
    document.addEventListener('click', (e) => {
      if (!this.$el.querySelector('.top-controls--left')?.contains(e.target)) {
        this.regionOpen = false
      }
      if (!this.$el.querySelector('.lang-dropdown:not(.top-controls--left .lang-dropdown)')?.contains(e.target)) {
        this.langOpen = false
      }
    })
  },
}
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background: #1a1a2e;
  height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #eaeaea;
  width: 100%;
  height: 100vh;
}

.screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 14px;
  padding: 20px;
  height: 100vh;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

h1 {
  font-size: 2.8rem;
  letter-spacing: 1px;
}

h2 {
  font-size: 1.8rem;
}

.subtitle {
  font-size: 1.1rem;
  color: #aab4c8;
  max-width: 400px;
}

.flag-container {
  position: relative;
  flex: 1;
  min-height: 0;
  width: 92vw;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #ffffff12;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 8px 32px #0005;
}

.flag-img {
  display: block;
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 6px;
}

.top-controls {
  position: fixed;
  top: 14px;
  right: 16px;
  z-index: 100;
  display: flex;
  align-items: center;
  gap: 8px;
}

.top-controls--left {
  right: auto;
  left: 16px;
}

.btn-topbar {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 14px;
  background: #0f3460cc;
  color: #e2e8f0;
  border: 1px solid #ffffff22;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
  backdrop-filter: blur(4px);
  white-space: nowrap;
}

.btn-topbar:hover {
  background: #0f3460;
  transform: translateY(-1px);
}

.lang-dropdown {
  position: relative;
}

.lang-menu {
  position: absolute;
  top: calc(100% + 6px);
  right: 0;
  background: #16213e;
  border: 1px solid #ffffff22;
  border-radius: 8px;
  overflow: hidden;
  min-width: 140px;
  box-shadow: 0 8px 24px #0006;
}

.lang-menu--left {
  right: auto;
  left: 0;
}

.lang-flag-img {
  width: 24px;
  height: auto;
  border-radius: 2px;
  display: inline-block;
  vertical-align: middle;
}

.lang-option {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
  padding: 10px 16px;
  background: none;
  color: #e2e8f0;
  border: none;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
  text-align: left;
}

.lang-option:hover {
  background: #0f3460;
}

.lang-option.active {
  background: #0f3460cc;
  color: #fff;
}

.flag-img--dimmed {
  filter: brightness(0.35);
}

.answer-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.answer-badge {
  font-size: 7rem;
  font-weight: bold;
  background: #16213ecc;
  border: 2px solid #e94560;
  border-radius: 10px;
  padding: 18px 40px;
  color: #fff;
  letter-spacing: 0.5px;
  width: 90%;
  backdrop-filter: blur(6px);
  box-shadow:
    0 0 12px #e9456066,
    0 0 32px #e9456033,
    0 0 64px #e9456018;
  animation: badgeGlow 0.5s ease-out;
}

@keyframes badgeGlow {
  from {
    opacity: 0;
    transform: scale(0.96);
    box-shadow: 0 0 0 #e94560;
  }
  to {
    opacity: 1;
    transform: scale(1);
    box-shadow:
      0 0 12px #e9456066,
      0 0 32px #e9456033,
      0 0 64px #e9456018;
  }
}

.action-row {
  display: flex;
  gap: 14px;
  flex-wrap: wrap;
  justify-content: center;
}

.btn {
  padding: 12px 28px;
  font-size: 1rem;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: transform 0.1s, opacity 0.2s;
}

.btn:hover {
  opacity: 0.88;
  transform: translateY(-1px);
}

.btn:active {
  transform: translateY(0);
}

.btn-primary {
  background: #e94560;
  color: #fff;
}

.btn-secondary {
  background: #0f3460;
  color: #e2e8f0;
}
</style>
