<script setup lang="ts">
import { computed, nextTick, onMounted, reactive, ref, watch } from 'vue'
import stadiumAsset from './assets/stadium-night.jpg'

type TeamShape = 'shield' | 'round' | 'diamond'
type TeamSlot = 'away' | 'home'

interface Team {
  id: string
  name: string
  fullName: string
  city: string
  abbr: string
  primary: string
  secondary: string
  shape: TeamShape
}

interface PosterTheme {
  id: string
  name: string
  accent: string
  accentSoft: string
  ink: string
}

const teams: Team[] = [
  { id: 'leones', name: 'Leones', fullName: 'Leones del Caracas', city: 'Caracas', abbr: 'LEO', primary: '#A4161A', secondary: '#F4C95D', shape: 'shield' },
  { id: 'navegantes', name: 'Navegantes', fullName: 'Navegantes del Magallanes', city: 'Magallanes', abbr: 'NAV', primary: '#1769AA', secondary: '#F4F7FB', shape: 'round' },
  { id: 'tiburones', name: 'Tiburones', fullName: 'Tiburones de La Guaira', city: 'La Guaira', abbr: 'TIB', primary: '#C62828', secondary: '#F7F8FA', shape: 'diamond' },
  { id: 'cardenales', name: 'Cardenales', fullName: 'Cardenales de Lara', city: 'Lara', abbr: 'CAR', primary: '#8E1921', secondary: '#E9C46A', shape: 'shield' },
  { id: 'tigres', name: 'Tigres', fullName: 'Tigres de Aragua', city: 'Aragua', abbr: 'TIG', primary: '#E76F2E', secondary: '#171E27', shape: 'round' },
  { id: 'aguilas', name: 'Águilas', fullName: 'Águilas del Zulia', city: 'Zulia', abbr: 'AGU', primary: '#D96C19', secondary: '#202A44', shape: 'diamond' },
  { id: 'caribes', name: 'Caribes', fullName: 'Caribes de Anzoátegui', city: 'Anzoátegui', abbr: 'CRI', primary: '#1C6E4A', secondary: '#F2A93B', shape: 'shield' },
  { id: 'bravos', name: 'Bravos', fullName: 'Bravos de Margarita', city: 'Margarita', abbr: 'BRV', primary: '#5A2A82', secondary: '#2BB3B1', shape: 'round' },
]

const themes: PosterTheme[] = [
  { id: 'lights', name: 'Luces', accent: '#F3A83B', accentSoft: '#FFD995', ink: '#07131D' },
  { id: 'classic', name: 'Clásico', accent: '#E6D3A3', accentSoft: '#FFF2CF', ink: '#10231F' },
  { id: 'heat', name: 'Fuego', accent: '#F06645', accentSoft: '#FFB29D', ink: '#1B0F12' },
]

const nextSaturday = () => {
  const date = new Date()
  const days = (6 - date.getDay() + 7) % 7 || 7
  date.setDate(date.getDate() + days)
  const year = date.getFullYear()
  const month = `${date.getMonth() + 1}`.padStart(2, '0')
  const day = `${date.getDate()}`.padStart(2, '0')
  return `${year}-${month}-${day}`
}

const posterSvg = ref<SVGSVGElement | null>(null)
const stadiumDataUrl = ref('')
const awayTeamId = ref('leones')
const homeTeamId = ref('navegantes')
const gameDate = ref(nextSaturday())
const gameTime = ref('19:00')
const venue = ref('Estadio Monumental Simón Bolívar')
const eyebrow = ref('TEMPORADA REGULAR')
const selectedThemeId = ref('lights')
const isExporting = ref(false)
const notice = ref('')
const noticeTimer = ref<number | undefined>()
const customLogos = reactive<Record<TeamSlot, string>>({ away: '', home: '' })
const logoUploadTokens = reactive<Record<TeamSlot, number>>({ away: 0, home: 0 })
let stadiumLoadPromise: Promise<string> | null = null

const awayTeam = computed(() => teams.find((team) => team.id === awayTeamId.value) ?? teams[0])
const homeTeam = computed(() => teams.find((team) => team.id === homeTeamId.value) ?? teams[1])
const selectedTheme = computed(() => themes.find((theme) => theme.id === selectedThemeId.value) ?? themes[0])
const hasTeamConflict = computed(() => awayTeamId.value === homeTeamId.value)
const isFormComplete = computed(() => Boolean(gameDate.value && gameTime.value && venue.value.trim() && !hasTeamConflict.value))

const posterTeams = computed(() => [
  { slot: 'away' as TeamSlot, label: 'VISITANTE', x: 282, team: awayTeam.value },
  { slot: 'home' as TeamSlot, label: 'LOCAL', x: 798, team: homeTeam.value },
])

const formattedDate = computed(() => {
  if (!gameDate.value) return 'FECHA POR CONFIRMAR'
  const date = new Date(`${gameDate.value}T12:00:00`)
  return new Intl.DateTimeFormat('es-VE', { weekday: 'long', day: '2-digit', month: 'long' })
    .format(date)
    .replace(',', ' ·')
    .toLocaleUpperCase('es-VE')
})

const gameYear = computed(() => gameDate.value ? new Date(`${gameDate.value}T12:00:00`).getFullYear() : new Date().getFullYear())

const formattedTime = computed(() => {
  if (!gameTime.value) return '--:--'
  const [hourValue, minutes = '00'] = gameTime.value.split(':')
  const hour = Number(hourValue)
  const suffix = hour >= 12 ? 'PM' : 'AM'
  return `${`${hour % 12 || 12}`.padStart(2, '0')}:${minutes} ${suffix}`
})

const venueDisplay = computed(() => venue.value.trim().toLocaleUpperCase('es-VE') || 'SEDE POR CONFIRMAR')
const venueLines = computed(() => {
  const value = venueDisplay.value
  if (value.length <= 38) return [value]
  const words = value.split(/\s+/)
  const firstLine: string[] = []
  const targetLength = Math.ceil(value.length / 2)
  while (words.length && `${firstLine.join(' ')} ${words[0]}`.trim().length <= targetLength + 3) {
    firstLine.push(words.shift() ?? '')
  }
  if (!firstLine.length) return [`${value.slice(0, 34)}…`]
  return [firstLine.join(' '), words.join(' ')].filter(Boolean)
})
const venueFontSize = computed(() => venueLines.value.length > 1 ? 23 : venueDisplay.value.length > 32 ? 25 : venueDisplay.value.length > 24 ? 29 : 34)
const eyebrowDisplay = computed(() => eyebrow.value.trim().toLocaleUpperCase('es-VE').slice(0, 28) || 'TEMPORADA REGULAR')

watch(awayTeamId, () => {
  logoUploadTokens.away += 1
  customLogos.away = ''
})
watch(homeTeamId, () => {
  logoUploadTokens.home += 1
  customLogos.home = ''
})

const showNotice = (message: string) => {
  notice.value = message
  if (noticeTimer.value) window.clearTimeout(noticeTimer.value)
  noticeTimer.value = window.setTimeout(() => { notice.value = '' }, 3200)
}

const swapTeams = () => {
  const previousAway = awayTeamId.value
  const previousAwayLogo = customLogos.away
  const previousHomeLogo = customLogos.home
  awayTeamId.value = homeTeamId.value
  homeTeamId.value = previousAway
  nextTick(() => {
    customLogos.away = previousHomeLogo
    customLogos.home = previousAwayLogo
  })
}

const handleLogoUpload = async (slot: TeamSlot, event: Event) => {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return
  const supportedTypes = new Set(['image/png', 'image/jpeg', 'image/webp', 'image/svg+xml'])
  if (!supportedTypes.has(file.type)) {
    showNotice('Selecciona un archivo de imagen válido.')
    input.value = ''
    return
  }
  if (file.size > 3 * 1024 * 1024) {
    showNotice('El logo debe pesar menos de 3 MB.')
    input.value = ''
    return
  }

  const uploadToken = ++logoUploadTokens[slot]
  const selectedTeamId = slot === 'away' ? awayTeamId.value : homeTeamId.value
  const sourceUrl = URL.createObjectURL(file)
  try {
    const image = new Image()
    image.decoding = 'async'
    image.src = sourceUrl
    await image.decode()
    if (!image.naturalWidth || !image.naturalHeight || image.naturalWidth * image.naturalHeight > 25_000_000) {
      throw new Error('Dimensiones de logo no válidas')
    }

    const canvas = document.createElement('canvas')
    canvas.width = 512
    canvas.height = 512
    const context = canvas.getContext('2d')
    if (!context) throw new Error('No se pudo procesar el logo')
    const availableSize = 464
    const scale = Math.min(availableSize / image.naturalWidth, availableSize / image.naturalHeight)
    const width = image.naturalWidth * scale
    const height = image.naturalHeight * scale
    context.drawImage(image, (512 - width) / 2, (512 - height) / 2, width, height)
    const normalizedLogo = canvas.toDataURL('image/png')

    const currentTeamId = slot === 'away' ? awayTeamId.value : homeTeamId.value
    if (uploadToken !== logoUploadTokens[slot] || currentTeamId !== selectedTeamId) return
    customLogos[slot] = normalizedLogo
    showNotice('Logo actualizado en la vista previa.')
  } catch (error) {
    console.error(error)
    if (uploadToken === logoUploadTokens[slot]) showNotice('No pudimos leer ese logo. Prueba con otro archivo.')
  } finally {
    URL.revokeObjectURL(sourceUrl)
    input.value = ''
  }
}

const removeCustomLogo = (slot: TeamSlot) => {
  logoUploadTokens[slot] += 1
  customLogos[slot] = ''
}

const loadStadiumAsDataUrl = async () => {
  if (stadiumDataUrl.value) return stadiumDataUrl.value
  if (!stadiumLoadPromise) {
    stadiumLoadPromise = (async () => {
      const response = await fetch(stadiumAsset)
      if (!response.ok) throw new Error('No se pudo cargar el fondo del estadio')
      const blob = await response.blob()
      stadiumDataUrl.value = await new Promise<string>((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = () => resolve(String(reader.result))
        reader.onerror = () => reject(reader.error)
        reader.readAsDataURL(blob)
      })
      return stadiumDataUrl.value
    })().catch((error) => {
      stadiumLoadPromise = null
      throw error
    })
  }
  return stadiumLoadPromise
}

const downloadPoster = async () => {
  if (!isFormComplete.value || !posterSvg.value) {
    showNotice(hasTeamConflict.value ? 'Elige dos equipos diferentes.' : 'Completa los datos del juego.')
    return
  }
  isExporting.value = true
  let svgUrl = ''
  try {
    await loadStadiumAsDataUrl()
    await nextTick()

    const svgClone = posterSvg.value.cloneNode(true) as SVGSVGElement
    svgClone.setAttribute('xmlns', 'http://www.w3.org/2000/svg')
    svgClone.setAttribute('width', '1080')
    svgClone.setAttribute('height', '1080')
    const serialized = new XMLSerializer().serializeToString(svgClone)
    svgUrl = URL.createObjectURL(new Blob([serialized], { type: 'image/svg+xml;charset=utf-8' }))
    const image = new Image()
    image.decoding = 'async'
    const imageReady = new Promise<void>((resolve, reject) => {
      image.onload = () => resolve()
      image.onerror = () => reject(new Error('No se pudo preparar la imagen'))
    })
    image.src = svgUrl
    await imageReady

    const canvas = document.createElement('canvas')
    canvas.width = 1080
    canvas.height = 1080
    const context = canvas.getContext('2d')
    if (!context) throw new Error('El navegador no permite exportar el cartel')
    context.drawImage(image, 0, 0, 1080, 1080)
    const pngBlob = await new Promise<Blob>((resolve, reject) => {
      canvas.toBlob((blob) => blob ? resolve(blob) : reject(new Error('No se pudo crear el PNG')), 'image/png')
    })
    const pngUrl = URL.createObjectURL(pngBlob)
    const link = document.createElement('a')
    link.href = pngUrl
    link.download = `${awayTeam.value.name}-vs-${homeTeam.value.name}.png`
    document.body.appendChild(link)
    link.click()
    link.remove()
    window.setTimeout(() => URL.revokeObjectURL(pngUrl), 1000)
    showNotice('Cartel descargado en alta resolución.')
  } catch (error) {
    console.error(error)
    showNotice('No pudimos exportar el cartel. Intenta de nuevo.')
  } finally {
    if (svgUrl) URL.revokeObjectURL(svgUrl)
    isExporting.value = false
  }
}

onMounted(() => {
  loadStadiumAsDataUrl().catch(() => { /* La vista previa conserva el recurso local. */ })
})
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <a class="brand" href="#" aria-label="Diamante, inicio">
        <span class="brand-mark" aria-hidden="true">
          <svg viewBox="0 0 40 40"><path d="M20 4 36 20 20 36 4 20Z" fill="none" stroke="currentColor" stroke-width="2.4" /><path d="M20 11 29 20 20 29 11 20Z" fill="currentColor" /><circle cx="20" cy="20" r="3.2" fill="#fff" /></svg>
        </span>
        <span><strong>Liga Deportiva Cristiana</strong><small>Jugando para Cristo</small></span>
      </a>
      <div class="topbar-meta"><span class="autosave" :class="{ invalid: !isFormComplete }"><i></i> {{ isFormComplete ? 'Diseño listo' : 'Completa los datos' }}</span><span class="resolution">1080 × 1080 px</span></div>
    </header>

    <main class="workspace">
      <aside class="control-panel">
        <div class="panel-intro">
          <span class="section-kicker">CREADOR DE CARTELES</span>
          <h1>Arma el próximo <em>juego.</em></h1>
          <p>Elige los equipos y descarga una pieza lista para publicar.</p>
        </div>

        <form class="match-form" @submit.prevent="downloadPoster">
          <section class="form-section">
            <div class="section-heading"><span>01</span><div><h2>El enfrentamiento</h2><p>Selecciona visitante y local</p></div></div>
            <div class="teams-control">
              <div class="team-field">
                <label for="away-team">Visitante</label>
                <div class="team-select-wrap">
                  <span class="mini-crest" :style="{ '--team-color': awayTeam.primary, '--team-accent': awayTeam.secondary }" aria-hidden="true">
                    <img v-if="customLogos.away" :src="customLogos.away" alt="" /><b v-else>{{ awayTeam.abbr.slice(0, 2) }}</b>
                  </span>
                  <select id="away-team" v-model="awayTeamId">
                    <option v-for="team in teams" :key="team.id" :value="team.id" :disabled="team.id === homeTeamId">{{ team.fullName }}</option>
                  </select>
                  <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6 8 4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.8" /></svg>
                </div>
                <div class="logo-actions">
                  <label class="upload-action"><input type="file" accept="image/png,image/jpeg,image/webp,image/svg+xml" :aria-label="`Subir logo del visitante, ${awayTeam.name}`" @change="handleLogoUpload('away', $event)" /><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 14V4m0 0L6.5 7.5M10 4l3.5 3.5M4 12.5V16h12v-3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round" /></svg>{{ customLogos.away ? 'Cambiar logo' : 'Subir logo' }}</label>
                  <button v-if="customLogos.away" type="button" class="clear-logo" :aria-label="`Quitar logo personalizado del visitante, ${awayTeam.name}`" @click="removeCustomLogo('away')">Quitar</button>
                </div>
              </div>

              <button class="swap-button" type="button" aria-label="Intercambiar equipos" title="Intercambiar equipos" @click="swapTeams"><svg viewBox="0 0 24 24" aria-hidden="true"><path d="M7 7h11m0 0-3-3m3 3-3 3M17 17H6m0 0 3 3m-3-3 3-3" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg></button>

              <div class="team-field">
                <label for="home-team">Local</label>
                <div class="team-select-wrap">
                  <span class="mini-crest" :style="{ '--team-color': homeTeam.primary, '--team-accent': homeTeam.secondary }" aria-hidden="true">
                    <img v-if="customLogos.home" :src="customLogos.home" alt="" /><b v-else>{{ homeTeam.abbr.slice(0, 2) }}</b>
                  </span>
                  <select id="home-team" v-model="homeTeamId">
                    <option v-for="team in teams" :key="team.id" :value="team.id" :disabled="team.id === awayTeamId">{{ team.fullName }}</option>
                  </select>
                  <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6 8 4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.8" /></svg>
                </div>
                <div class="logo-actions">
                  <label class="upload-action"><input type="file" accept="image/png,image/jpeg,image/webp,image/svg+xml" :aria-label="`Subir logo del local, ${homeTeam.name}`" @change="handleLogoUpload('home', $event)" /><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 14V4m0 0L6.5 7.5M10 4l3.5 3.5M4 12.5V16h12v-3.5" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round" /></svg>{{ customLogos.home ? 'Cambiar logo' : 'Subir logo' }}</label>
                  <button v-if="customLogos.home" type="button" class="clear-logo" :aria-label="`Quitar logo personalizado del local, ${homeTeam.name}`" @click="removeCustomLogo('home')">Quitar</button>
                </div>
              </div>
            </div>
            <p v-if="hasTeamConflict" class="field-error">Elige dos equipos diferentes.</p>
          </section>

          <section class="form-section">
            <div class="section-heading"><span>02</span><div><h2>Datos del juego</h2><p>Define cuándo y dónde</p></div></div>
            <div class="field-grid">
              <label class="input-field"><span>Fecha</span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M5 3v3m10-3v3M3.5 8h13M4 5h12a1 1 0 0 1 1 1v10H3V6a1 1 0 0 1 1-1Z" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" /></svg><input v-model="gameDate" type="date" required /></span></label>
              <label class="input-field"><span>Hora</span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><circle cx="10" cy="10" r="7" fill="none" stroke="currentColor" stroke-width="1.5" /><path d="M10 6v4l2.8 1.7" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" /></svg><input v-model="gameTime" type="time" required /></span></label>
              <label class="input-field field-wide"><span>Estadio</span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 17s5-4.6 5-9a5 5 0 1 0-10 0c0 4.4 5 9 5 9Z" fill="none" stroke="currentColor" stroke-width="1.5" /><circle cx="10" cy="8" r="1.8" fill="none" stroke="currentColor" stroke-width="1.5" /></svg><input v-model="venue" list="venues" maxlength="54" placeholder="Nombre del estadio" required /></span></label>
              <datalist id="venues"><option value="Estadio Monumental Simón Bolívar" /><option value="Estadio Universitario de Caracas" /><option value="José Bernardo Pérez" /><option value="Antonio Herrera Gutiérrez" /><option value="Luis Aparicio El Grande" /></datalist>
              <label class="input-field field-wide"><span>Etiqueta <small>Opcional</small></span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M4 4h8l4 4-8 8-4-4V4Z" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round" /><circle cx="8" cy="8" r="1.2" fill="currentColor" /></svg><input v-model="eyebrow" maxlength="28" placeholder="Temporada regular" /></span></label>
            </div>
          </section>

          <section class="form-section compact-section">
            <div class="section-heading theme-heading"><span>03</span><div><h2>Color de acento</h2><p>El toque final del cartel</p></div></div>
            <div class="theme-options" role="radiogroup" aria-label="Color de acento">
              <label v-for="theme in themes" :key="theme.id" :class="{ active: selectedThemeId === theme.id }"><input v-model="selectedThemeId" type="radio" name="theme" :value="theme.id" /><i :style="{ background: theme.accent }"></i><span>{{ theme.name }}</span><svg viewBox="0 0 16 16" aria-hidden="true"><path d="m4 8 2.5 2.5L12 5" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg></label>
            </div>
          </section>

          <div class="form-actions">
            <button class="download-button" type="submit" :disabled="isExporting"><svg v-if="!isExporting" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 4v11m0 0 4-4m-4 4-4-4M5 19h14" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg><span v-else class="button-spinner" aria-hidden="true"></span>{{ isExporting ? 'Preparando cartel…' : 'Descargar cartel PNG' }}</button>
            <p><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M8 1.5 13.5 4v3.8c0 3.3-2.3 5.6-5.5 6.7-3.2-1.1-5.5-3.4-5.5-6.7V4L8 1.5Z" fill="none" stroke="currentColor" stroke-width="1.2" /><path d="m5.5 8 1.6 1.6 3.5-3.5" fill="none" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" /></svg> Se procesa en tu navegador. Nada se publica.</p>
          </div>
        </form>
      </aside>

      <section class="preview-panel" aria-labelledby="preview-title">
        <div class="preview-toolbar"><div><span class="live-dot"></span><div><h2 id="preview-title">Vista previa</h2><p>Se actualiza en tiempo real</p></div></div><span class="format-pill">PUBLICACIÓN · 1:1</span></div>
        <div class="poster-stage">
          <div class="poster-shadow" aria-hidden="true"></div>
          <svg ref="posterSvg" class="poster" viewBox="0 0 1080 1080" role="img" :aria-label="`${awayTeam.fullName} versus ${homeTeam.fullName}, ${formattedDate}, ${formattedTime}, ${venueDisplay}`" xmlns="http://www.w3.org/2000/svg">
            <defs>
              <linearGradient id="posterShade" x1="0" y1="0" x2="0" y2="1"><stop offset="0" :stop-color="selectedTheme.ink" stop-opacity=".76" /><stop offset=".48" stop-color="#07131D" stop-opacity=".68" /><stop offset="1" stop-color="#03090D" stop-opacity=".97" /></linearGradient>
              <linearGradient id="cardFill" x1="0" y1="0" x2="0" y2="1"><stop offset="0" stop-color="#10232C" stop-opacity=".89" /><stop offset="1" stop-color="#071219" stop-opacity=".94" /></linearGradient>
              <radialGradient id="stadiumGlow" cx="50%" cy="35%" r="58%"><stop offset="0" :stop-color="selectedTheme.accent" stop-opacity=".14" /><stop offset="1" stop-color="#000000" stop-opacity="0" /></radialGradient>
              <linearGradient id="accentLine" x1="0" y1="0" x2="1" y2="0"><stop offset="0" :stop-color="selectedTheme.accent" stop-opacity="0" /><stop offset=".5" :stop-color="selectedTheme.accent" /><stop offset="1" :stop-color="selectedTheme.accent" stop-opacity="0" /></linearGradient>
              <pattern id="dotGrid" width="24" height="24" patternUnits="userSpaceOnUse"><circle cx="2" cy="2" r="1.2" fill="#FFFFFF" fill-opacity=".09" /></pattern>
              <filter id="softShadow" x="-30%" y="-30%" width="160%" height="170%"><feDropShadow dx="0" dy="18" stdDeviation="18" flood-color="#000000" flood-opacity=".42" /></filter>
              <filter id="logoShadow" x="-30%" y="-30%" width="160%" height="170%"><feDropShadow dx="0" dy="12" stdDeviation="10" flood-color="#000000" flood-opacity=".5" /></filter>
              <clipPath id="logo-away"><circle cx="0" cy="0" r="91" /></clipPath><clipPath id="logo-home"><circle cx="0" cy="0" r="91" /></clipPath>
            </defs>

            <rect width="1080" height="1080" fill="#061018" />
            <image :href="stadiumDataUrl || stadiumAsset" width="1080" height="1080" preserveAspectRatio="xMidYMid slice" />
            <rect width="1080" height="1080" fill="url(#posterShade)" /><rect width="1080" height="1080" fill="url(#stadiumGlow)" /><rect width="1080" height="1080" fill="url(#dotGrid)" opacity=".26" />
            <g opacity=".24" fill="none" :stroke="selectedTheme.accent" stroke-width="2"><path d="M540 355 743 558 540 761 337 558Z" /><path d="M540 400 698 558 540 716 382 558Z" stroke-dasharray="7 13" /><circle cx="540" cy="558" r="7" :fill="selectedTheme.accent" stroke="none" /></g>

            <g font-family="Arial, Helvetica, sans-serif">
              <g transform="translate(72 58)"><path d="M20 0 40 20 20 40 0 20Z" fill="none" :stroke="selectedTheme.accent" stroke-width="2" /><path d="M20 9 31 20 20 31 9 20Z" :fill="selectedTheme.accent" /><circle cx="20" cy="20" r="4" fill="#07131D" /><text x="56" y="17" fill="#FFFFFF" font-size="23" font-weight="800" letter-spacing="3">DIAMANTE</text><text x="56" y="37" fill="#FFFFFF" fill-opacity=".52" font-size="11" font-weight="600" letter-spacing="3.2">ESTUDIO DE JUEGO</text></g>
              <g transform="translate(1008 61)" text-anchor="end"><text y="15" fill="#FFFFFF" fill-opacity=".54" font-size="11" font-weight="700" letter-spacing="2.5">JORNADA DE JUEGO</text><text y="38" fill="#FFFFFF" font-size="18" font-weight="800" letter-spacing="2">{{ gameYear }}</text></g>
              <path d="M72 122H1008" stroke="#FFFFFF" stroke-opacity=".17" /><path d="M72 122H245" :stroke="selectedTheme.accent" stroke-width="3" />
              <text x="540" y="174" text-anchor="middle" :fill="selectedTheme.accentSoft" font-size="15" font-weight="800" letter-spacing="6.2">{{ eyebrowDisplay }}</text>
              <text x="540" y="229" text-anchor="middle" fill="#FFFFFF" font-size="53" font-weight="900" letter-spacing="5">PRÓXIMO JUEGO</text>
              <text x="540" y="267" text-anchor="middle" fill="#FFFFFF" fill-opacity=".68" font-size="17" font-weight="700" letter-spacing="2.7">{{ formattedDate }}</text>

              <g filter="url(#softShadow)"><rect x="72" y="302" width="936" height="496" rx="26" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" /><path d="M72 363H1008" stroke="#FFFFFF" stroke-opacity=".12" /><path d="M72 731H1008" stroke="#FFFFFF" stroke-opacity=".12" /><path d="M540 363V731" stroke="#FFFFFF" stroke-opacity=".07" /><path d="M72 302H1008" :stroke="selectedTheme.accent" stroke-width="3" opacity=".9" /></g>

              <g v-for="item in posterTeams" :key="item.slot" :transform="`translate(${item.x} 0)`">
                <text x="0" y="341" text-anchor="middle" fill="#FFFFFF" fill-opacity=".48" font-size="13" font-weight="800" letter-spacing="4.5">{{ item.label }}</text>
                <g transform="translate(0 501)" filter="url(#logoShadow)">
                  <circle r="111" fill="#02080C" fill-opacity=".8" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" /><circle r="99" :fill="item.team.primary" /><circle r="91" fill="#FFFFFF" fill-opacity=".07" />
                  <template v-if="!customLogos[item.slot]">
                    <path v-if="item.team.shape === 'shield'" d="M-64-62H64V20C64 61 24 82 0 93-24 82-64 61-64 20Z" fill="#07131D" fill-opacity=".34" :stroke="item.team.secondary" stroke-width="5" />
                    <g v-else-if="item.team.shape === 'round'"><circle r="69" fill="#07131D" fill-opacity=".3" :stroke="item.team.secondary" stroke-width="6" /><circle r="57" fill="none" :stroke="item.team.secondary" stroke-opacity=".42" stroke-width="2" stroke-dasharray="5 7" /></g>
                    <g v-else transform="rotate(45)"><rect x="-57" y="-57" width="114" height="114" rx="16" fill="#07131D" fill-opacity=".32" :stroke="item.team.secondary" stroke-width="6" /></g>
                    <path d="M-76-75Q0-38 76-75" fill="none" :stroke="item.team.secondary" stroke-opacity=".35" stroke-width="3" /><path d="M-76 75Q0 38 76 75" fill="none" :stroke="item.team.secondary" stroke-opacity=".35" stroke-width="3" />
                    <text y="7" text-anchor="middle" dominant-baseline="middle" :fill="item.team.secondary" font-size="48" font-weight="900" letter-spacing="-1">{{ item.team.abbr }}</text>
                  </template>
                  <image v-else :href="customLogos[item.slot]" x="-91" y="-91" width="182" height="182" preserveAspectRatio="xMidYMid meet" :clip-path="`url(#logo-${item.slot})`" />
                </g>
                <text x="0" y="658" text-anchor="middle" fill="#FFFFFF" font-size="42" font-weight="900" letter-spacing="1">{{ item.team.name.toLocaleUpperCase('es-VE') }}</text>
                <text x="0" y="689" text-anchor="middle" :fill="selectedTheme.accentSoft" font-size="15" font-weight="700" letter-spacing="4.5">{{ item.team.city.toLocaleUpperCase('es-VE') }}</text>
              </g>

              <g transform="translate(540 500)"><circle r="53" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" /><circle r="43" :fill="selectedTheme.accent" /><path d="M-29-24Q0-4 29-24M-29 24Q0 4 29 24" fill="none" stroke="#07131D" stroke-opacity=".45" stroke-width="2.5" /><text y="7" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="27" font-weight="900" letter-spacing="-1">VS</text></g>
              <g transform="translate(120 752)"><circle cx="13" cy="13" r="13" :fill="selectedTheme.accent" fill-opacity=".15" /><path d="M13 5.5c-3.4 0-6 2.6-6 6 0 5 6 9.5 6 9.5s6-4.5 6-9.5c0-3.4-2.6-6-6-6Zm0 8.2a2.3 2.3 0 1 1 0-4.6 2.3 2.3 0 0 1 0 4.6Z" :fill="selectedTheme.accent" /><text x="39" y="18" fill="#FFFFFF" fill-opacity=".72" font-size="13" font-weight="700" letter-spacing="1.2">VENEZUELA · {{ gameYear }}</text></g>

              <g transform="translate(72 844)"><rect width="936" height="142" rx="22" fill="#050D12" fill-opacity=".9" stroke="#FFFFFF" stroke-opacity=".13" /><rect width="8" height="142" rx="4" :fill="selectedTheme.accent" />
                <g transform="translate(47 34)"><path d="M18 0c-9.5 0-17 7.4-17 17 0 14.2 17 27 17 27s17-12.8 17-27C35 7.4 27.5 0 18 0Zm0 23.2a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13Z" :fill="selectedTheme.accent" /><text x="55" y="10" fill="#FFFFFF" fill-opacity=".46" font-size="12" font-weight="800" letter-spacing="3.2">SEDE</text><text x="55" fill="#FFFFFF" :font-size="venueFontSize" font-weight="900" letter-spacing=".3"><tspan v-for="(line, index) in venueLines" :key="line" x="55" :y="venueLines.length > 1 ? 37 + index * 28 : 47">{{ line }}</tspan></text></g>
                <path d="M720 24V118" stroke="#FFFFFF" stroke-opacity=".13" /><g transform="translate(828 25)" text-anchor="middle"><circle cy="17" r="17" fill="none" :stroke="selectedTheme.accent" stroke-width="2.5" /><path d="M0 7v11l7 4" fill="none" :stroke="selectedTheme.accent" stroke-width="2.5" stroke-linecap="round" /><text y="65" fill="#FFFFFF" fill-opacity=".46" font-size="10" font-weight="800" letter-spacing="2">PRIMER PITCHEO</text><text y="104" fill="#FFFFFF" font-size="29" font-weight="900" letter-spacing=".2">{{ formattedTime }}</text></g>
              </g>

              <g transform="translate(72 1030)"><text fill="#FFFFFF" fill-opacity=".42" font-size="11" font-weight="700" letter-spacing="2.4">BÉISBOL · PASIÓN · TRADICIÓN</text><text x="936" text-anchor="end" fill="#FFFFFF" fill-opacity=".42" font-size="11" font-weight="700" letter-spacing="2.4">#DÍADEJUEGO</text></g>
              <rect x="0" y="1071" width="1080" height="9" :fill="selectedTheme.accent" /><rect x="238" y="1071" width="604" height="9" fill="url(#accentLine)" />
            </g>
          </svg>
        </div>
        <div class="preview-footnote"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M3 5.5h14v9H3zM6 3v2.5M14 3v2.5M6 14.5V17h8v-2.5" fill="none" stroke="currentColor" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round" /></svg><span><strong>Alta resolución.</strong> El archivo se exportará en formato PNG de 1080 × 1080 px.</span></div>
      </section>
    </main>

    <Transition name="toast"><div v-if="notice" class="toast-message" role="status"><svg viewBox="0 0 20 20" aria-hidden="true"><circle cx="10" cy="10" r="8" fill="none" stroke="currentColor" stroke-width="1.5" /><path d="m6.5 10 2.2 2.2 4.8-4.8" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round" /></svg>{{ notice }}</div></Transition>
  </div>
</template>
