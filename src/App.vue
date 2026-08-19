<script setup lang="ts">
import { computed, nextTick, onMounted, reactive, ref } from 'vue'
import stadiumAsset from './assets/stadium-night.jpg'
import bgStadiumNight from './assets/backgrounds/stadium-night.jpg'
import bgSoftballDay from './assets/backgrounds/softball-day.jpg'
import bgStadiumLights from './assets/backgrounds/stadium-lights.jpg'
import bgStadiumSunset from './assets/backgrounds/stadium-sunset.jpg'
import logoLigaAsset from './assets/Logos/Logo de softball de la liga cristiana.png'
import logoCristoElSalvador from './assets/Logos/optimized/cristo-el-salvador.png'
import logoDiosEsBueno from './assets/Logos/optimized/dios-es-bueno.png'
import logoFraternidadCristiana from './assets/Logos/optimized/fraternidad-cristiana.png'
import logoIdp from './assets/Logos/optimized/idp.png'
import logoJuan316 from './assets/Logos/optimized/juan-3-16.png'
import logoLaRed from './assets/Logos/optimized/la-red.png'
import logoLuzYVida from './assets/Logos/optimized/luz-y-vida.png'

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
  logo: string
}

interface Match {
  id: string
  awayTeamId: string
  homeTeamId: string
  time: string
}

interface PosterTheme {
  id: string
  name: string
  accent: string
  accentSoft: string
  ink: string
}

interface PosterBg {
  id: string
  name: string
  asset: string
}

const teams: Team[] = [
  { id: 'cristo-el-salvador', name: 'Cristo El Salvador', fullName: 'Cristo El Salvador', city: 'Liga Cristiana', abbr: 'CES', primary: '#0C4A6E', secondary: '#38BDF8', shape: 'shield', logo: logoCristoElSalvador },
  { id: 'dios-es-bueno', name: 'Dios es Bueno', fullName: 'Dios es Bueno', city: 'Liga Cristiana', abbr: 'DEB', primary: '#D97706', secondary: '#FEF3C7', shape: 'round', logo: logoDiosEsBueno },
  { id: 'fraternidad-cristiana', name: 'Fraternidad Cristiana', fullName: 'Fraternidad Cristiana', city: 'Liga Cristiana', abbr: 'FC', primary: '#1E40AF', secondary: '#93C5FD', shape: 'shield', logo: logoFraternidadCristiana },
  { id: 'idp', name: 'IDP', fullName: 'IDP', city: 'Liga Cristiana', abbr: 'IDP', primary: '#2563EB', secondary: '#EF4444', shape: 'diamond', logo: logoIdp },
  { id: 'juan-3-16', name: 'Juan 3:16', fullName: 'Juan 3:16', city: 'Liga Cristiana', abbr: 'J3:16', primary: '#991B1B', secondary: '#FDE047', shape: 'round', logo: logoJuan316 },
  { id: 'la-red', name: 'La Red', fullName: 'La Red', city: 'Liga Cristiana', abbr: 'RED', primary: '#DC2626', secondary: '#1E293B', shape: 'shield', logo: logoLaRed },
  { id: 'luz-y-vida', name: 'Luz y Vida', fullName: 'Luz y Vida', city: 'Liga Cristiana', abbr: 'LYV', primary: '#16A34A', secondary: '#FEF08A', shape: 'diamond', logo: logoLuzYVida },
]

const themes: PosterTheme[] = [
  { id: 'lights', name: 'Luces', accent: '#F3A83B', accentSoft: '#FFD995', ink: '#07131D' },
  { id: 'classic', name: 'Clásico', accent: '#E6D3A3', accentSoft: '#FFF2CF', ink: '#10231F' },
  { id: 'heat', name: 'Fuego', accent: '#F06645', accentSoft: '#FFB29D', ink: '#1B0F12' },
]

const backgrounds: PosterBg[] = [
  { id: 'night', name: 'Estadio Noche', asset: bgStadiumNight },
  { id: 'day', name: 'Campo de Día', asset: bgSoftballDay },
  { id: 'lights', name: 'Reflectores Pro', asset: bgStadiumLights },
  { id: 'sunset', name: 'Atardecer', asset: bgStadiumSunset },
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
const leagueLogoDataUrl = ref('')
const teamDataUrls = reactive<Record<string, string>>({})
const bgDataUrls = reactive<Record<string, string>>({})

const matches = reactive<Match[]>([
  { id: 'm1', awayTeamId: 'cristo-el-salvador', homeTeamId: 'dios-es-bueno', time: '19:00' },
])

const gameDate = ref(nextSaturday())
const venue = ref('Estadio Felipe Rivas')
const eyebrow = ref('TEMPORADA REGULAR')
const posterTitle = ref('')
const selectedThemeId = ref('lights')
const selectedBgId = ref('night')
const isExporting = ref(false)
const notice = ref('')
const noticeTimer = ref<number | undefined>()

const getTeam = (id: string): Team => {
  return teams.find((team) => team.id === id) ?? teams[0]
}

const selectedTheme = computed(() => themes.find((theme) => theme.id === selectedThemeId.value) ?? themes[0])
const currentBg = computed(() => backgrounds.find((b) => b.id === selectedBgId.value) ?? backgrounds[0])

const hasAnyConflict = computed(() => {
  return matches.some((m) => m.awayTeamId === m.homeTeamId)
})

const isFormComplete = computed(() => {
  return Boolean(gameDate.value && venue.value.trim() && !hasAnyConflict.value && matches.length > 0)
})

const posterTeamsSingle = computed(() => [
  { slot: 'away' as TeamSlot, label: 'VISITANTE', x: 282, team: getTeam(matches[0].awayTeamId) },
  { slot: 'home' as TeamSlot, label: 'LOCAL', x: 798, team: getTeam(matches[0].homeTeamId) },
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

const formatTime = (timeStr: string) => {
  if (!timeStr) return '--:--'
  const [hourValue, minutes = '00'] = timeStr.split(':')
  const hour = Number(hourValue)
  const suffix = hour >= 12 ? 'PM' : 'AM'
  return `${`${hour % 12 || 12}`.padStart(2, '0')}:${minutes} ${suffix}`
}

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
const computedPosterTitle = computed(() => {
  if (posterTitle.value.trim()) return posterTitle.value.trim().toLocaleUpperCase('es-VE')
  return matches.length > 1 ? 'CARTELERA DE JUEGOS' : 'PRÓXIMO JUEGO'
})

// Format team name dynamically for best typography in the SVG poster
const formatTeamName = (name: string) => {
  const upper = name.toLocaleUpperCase('es-VE').trim()
  if (upper === 'FRATERNIDAD CRISTIANA') {
    return {
      lines: ['FRATERNIDAD', 'CRISTIANA'],
      fontSize: 27,
      startY: 642,
      lineHeight: 31,
      cityY: 704,
    }
  }
  if (upper === 'CRISTO EL SALVADOR') {
    return {
      lines: ['CRISTO EL', 'SALVADOR'],
      fontSize: 28,
      startY: 642,
      lineHeight: 31,
      cityY: 704,
    }
  }
  if (upper === 'DIOS ES BUENO') {
    return {
      lines: ['DIOS ES', 'BUENO'],
      fontSize: 30,
      startY: 642,
      lineHeight: 32,
      cityY: 704,
    }
  }
  if (upper === 'LUZ Y VIDA') {
    return {
      lines: ['LUZ Y VIDA'],
      fontSize: 36,
      startY: 658,
      lineHeight: 0,
      cityY: 690,
    }
  }
  if (upper.length <= 10) {
    return {
      lines: [upper],
      fontSize: upper.length <= 4 ? 42 : 38,
      startY: 658,
      lineHeight: 0,
      cityY: 690,
    }
  }
  const words = upper.split(' ')
  if (words.length >= 2) {
    const mid = Math.ceil(words.length / 2)
    return {
      lines: [words.slice(0, mid).join(' '), words.slice(mid).join(' ')],
      fontSize: 28,
      startY: 642,
      lineHeight: 31,
      cityY: 704,
    }
  }
  return {
    lines: [upper],
    fontSize: 32,
    startY: 658,
    lineHeight: 0,
    cityY: 690,
  }
}

const formatMultiName = (name: string, layout: 'two' | 'three' | 'four') => {
  const upper = name.toLocaleUpperCase('es-VE').trim()
  if (layout === 'two') {
    if (upper === 'FRATERNIDAD CRISTIANA') {
      return { lines: ['FRATERNIDAD', 'CRISTIANA'], fontSize: 20, startY: -10, lineHeight: 22, labelY: 34 }
    }
    if (upper === 'CRISTO EL SALVADOR') {
      return { lines: ['CRISTO EL', 'SALVADOR'], fontSize: 21, startY: -10, lineHeight: 22, labelY: 34 }
    }
    if (upper === 'DIOS ES BUENO') {
      return { lines: ['DIOS ES', 'BUENO'], fontSize: 22, startY: -10, lineHeight: 23, labelY: 34 }
    }
    if (upper.length > 12) {
      const words = upper.split(' ')
      const mid = Math.ceil(words.length / 2)
      return { lines: [words.slice(0, mid).join(' '), words.slice(mid).join(' ')], fontSize: 20, startY: -10, lineHeight: 22, labelY: 34 }
    }
    return { lines: [upper], fontSize: 24, startY: 0, lineHeight: 0, labelY: 23 }
  }

  if (layout === 'three') {
    if (upper === 'FRATERNIDAD CRISTIANA') {
      return { lines: ['FRATERNIDAD', 'CRISTIANA'], fontSize: 20, startY: -3, lineHeight: 22 }
    }
    if (upper === 'CRISTO EL SALVADOR') {
      return { lines: ['CRISTO EL', 'SALVADOR'], fontSize: 21, startY: -3, lineHeight: 22 }
    }
    if (upper === 'DIOS ES BUENO') {
      return { lines: ['DIOS ES', 'BUENO'], fontSize: 22, startY: -3, lineHeight: 23 }
    }
    if (upper.length > 12) {
      const words = upper.split(' ')
      const mid = Math.ceil(words.length / 2)
      return { lines: [words.slice(0, mid).join(' '), words.slice(mid).join(' ')], fontSize: 20, startY: -3, lineHeight: 22 }
    }
    return { lines: [upper], fontSize: 26, startY: 8, lineHeight: 0 }
  }

  // layout === 'four'
  if (upper === 'FRATERNIDAD CRISTIANA') {
    return { lines: ['FRATERNIDAD', 'CRISTIANA'], fontSize: 17, startY: -3, lineHeight: 18 }
  }
  if (upper === 'CRISTO EL SALVADOR') {
    return { lines: ['CRISTO EL', 'SALVADOR'], fontSize: 18, startY: -3, lineHeight: 19 }
  }
  if (upper === 'DIOS ES BUENO') {
    return { lines: ['DIOS ES', 'BUENO'], fontSize: 19, startY: -3, lineHeight: 20 }
  }
  if (upper.length > 12) {
    const words = upper.split(' ')
    const mid = Math.ceil(words.length / 2)
    return { lines: [words.slice(0, mid).join(' '), words.slice(mid).join(' ')], fontSize: 17, startY: -3, lineHeight: 18 }
  }
  return { lines: [upper], fontSize: 22, startY: 8, lineHeight: 0 }
}

const showNotice = (message: string) => {
  notice.value = message
  if (noticeTimer.value) window.clearTimeout(noticeTimer.value)
  noticeTimer.value = window.setTimeout(() => { notice.value = '' }, 3200)
}

const addMatch = () => {
  if (matches.length >= 4) {
    showNotice('Máximo 4 encuentros por cartel diario.')
    return
  }
  const defaultTimes = ['09:00', '11:30', '14:00', '16:30']
  const nextTime = defaultTimes[matches.length] || '14:00'
  const usedTeams = new Set(matches.flatMap((m) => [m.awayTeamId, m.homeTeamId]))
  const available = teams.filter((t) => !usedTeams.has(t.id))
  const awayId = available[0]?.id || teams[(matches.length * 2) % teams.length].id
  const homeId = available[1]?.id || teams[(matches.length * 2 + 1) % teams.length].id

  matches.push({
    id: `m_${Date.now()}_${Math.random().toString(36).slice(2, 6)}`,
    awayTeamId: awayId,
    homeTeamId: homeId,
    time: nextTime,
  })
  showNotice(`Encuentro #${matches.length} añadido.`)
}

const removeMatch = (index: number) => {
  if (matches.length <= 1) {
    showNotice('Debe haber al menos un encuentro.')
    return
  }
  matches.splice(index, 1)
  showNotice('Encuentro eliminado.')
}

const swapTeams = (match: Match) => {
  const previousAway = match.awayTeamId
  match.awayTeamId = match.homeTeamId
  match.homeTeamId = previousAway
}

const urlToDataUrl = async (url: string): Promise<string> => {
  try {
    const response = await fetch(url)
    if (!response.ok) throw new Error('Error al cargar imagen')
    const blob = await response.blob()
    return await new Promise<string>((resolve, reject) => {
      const reader = new FileReader()
      reader.onload = () => resolve(String(reader.result))
      reader.onerror = () => reject(reader.error)
      reader.readAsDataURL(blob)
    })
  } catch {
    return url
  }
}

const preloadAssets = async () => {
  try {
    const [stadium, league] = await Promise.all([
      urlToDataUrl(stadiumAsset),
      urlToDataUrl(logoLigaAsset),
    ])
    stadiumDataUrl.value = stadium
    leagueLogoDataUrl.value = league

    for (const team of teams) {
      teamDataUrls[team.id] = await urlToDataUrl(team.logo)
    }

    for (const bg of backgrounds) {
      bgDataUrls[bg.id] = await urlToDataUrl(bg.asset)
    }
  } catch (e) {
    console.warn('Error precargando imágenes:', e)
  }
}

const downloadPoster = async () => {
  if (!isFormComplete.value || !posterSvg.value) {
    showNotice(hasAnyConflict.value ? 'Elige equipos diferentes en cada juego.' : 'Completa los datos del juego.')
    return
  }
  isExporting.value = true
  let svgUrl = ''
  try {
    await preloadAssets()
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
    let dateSuffix = 'fecha'
    if (gameDate.value) {
      const date = new Date(`${gameDate.value}T12:00:00`)
      const weekday = new Intl.DateTimeFormat('es-VE', { weekday: 'long' }).format(date)
      const cleanWeekday = weekday.normalize('NFD').replace(/[\u0300-\u036f]/g, '').toLowerCase()
      const day = `${date.getDate()}`.padStart(2, '0')
      dateSuffix = `${cleanWeekday}-${day}`
    }
    link.download = `softball-liga-cristiana-${dateSuffix}.png`
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
  preloadAssets()
})
</script>

<template>
  <div class="app-shell">
    <header class="topbar">
      <a class="brand" href="#" aria-label="Liga Deportiva Cristiana, inicio">
        <span class="brand-mark brand-league-mark" aria-hidden="true">
          <img :src="leagueLogoDataUrl || logoLigaAsset" alt="Liga Deportiva Cristiana" />
        </span>
        <span><strong>Liga Deportiva Cristiana</strong><small>Jugando para Cristo</small></span>
      </a>
      <div class="topbar-meta"><span class="autosave" :class="{ invalid: !isFormComplete }"><i></i> {{ isFormComplete ? 'Diseño listo' : 'Completa los datos' }}</span><span class="resolution">1080 × 1080 px</span></div>
    </header>

    <main class="workspace">
      <aside class="control-panel">
        <div class="panel-intro">
          <span class="section-kicker">CREADOR DE JORNADAS</span>
          <h1>Arma el próximo <em>juego.</em></h1>
          <p>Elige los equipos y descarga una pieza lista para publicar.</p>
        </div>

        <form class="match-form" @submit.prevent="downloadPoster">
          <!-- SECTION 01: ENCOUNTERS -->
          <section class="form-section">
            <div class="section-heading-row">
              <div class="section-heading">
                <span>01</span>
                <div>
                  <h2>{{ matches.length === 1 ? 'El enfrentamiento' : 'Los enfrentamientos' }}</h2>
                  <p>{{ matches.length === 1 ? 'Selecciona visitante y local' : `${matches.length} juegos programados (máx. 4)` }}</p>
                </div>
              </div>
              <button
                v-if="matches.length < 4"
                type="button"
                class="add-match-btn"
                @click="addMatch"
                title="Añadir otro juego a la jornada"
              >
                <svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 4v12m-6-6h12" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" /></svg>
                <span>Añadir encuentro</span>
              </button>
            </div>

            <!-- ANIMATED MATCHES STACK -->
            <TransitionGroup name="match-card" tag="div" class="matches-stack">
              <div
                v-for="(match, mIdx) in matches"
                :key="match.id"
                class="match-card-box"
              >
                <!-- TOP HEADER BAR (CONSISTENT & STABLE) -->
                <div class="match-box-top">
                  <div class="match-box-top-left">
                    <span class="match-badge-tag">Juego {{ mIdx + 1 }}</span>
                    <label class="match-time-input-wrap">
                      <span>Hora:</span>
                      <input v-model="match.time" type="time" required />
                    </label>
                  </div>
                  <button
                    v-if="matches.length > 1"
                    type="button"
                    class="remove-match-btn"
                    @click="removeMatch(mIdx)"
                    title="Eliminar este juego"
                    aria-label="Eliminar este juego"
                  >
                    <svg viewBox="0 0 24 24" aria-hidden="true">
                      <path d="M4 7h16M10 11v6M14 11v6M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2l1-12M9 7V4a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v3" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
                    </svg>
                  </button>
                </div>

                <div class="teams-control">
                  <!-- AWAY TEAM -->
                  <div class="team-field">
                    <label :for="`away-team-${mIdx}`">Visitante</label>
                    <div class="team-select-wrap">
                      <span class="mini-crest" :style="{ '--team-color': getTeam(match.awayTeamId).primary, '--team-accent': getTeam(match.awayTeamId).secondary }" aria-hidden="true">
                        <img :src="teamDataUrls[match.awayTeamId] || getTeam(match.awayTeamId).logo" :alt="getTeam(match.awayTeamId).name" />
                      </span>
                      <select :id="`away-team-${mIdx}`" v-model="match.awayTeamId">
                        <option v-for="team in teams" :key="team.id" :value="team.id" :disabled="team.id === match.homeTeamId">{{ team.fullName }}</option>
                      </select>
                      <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6 8 4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.8" /></svg>
                    </div>
                  </div>

                  <!-- SWAP BUTTON -->
                  <button class="swap-button" type="button" aria-label="Intercambiar equipos" title="Intercambiar equipos" @click="swapTeams(match)">
                    <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M7 7h11m0 0-3-3m3 3-3 3M17 17H6m0 0 3 3m-3-3 3-3" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
                  </button>

                  <!-- HOME TEAM -->
                  <div class="team-field">
                    <label :for="`home-team-${mIdx}`">Local</label>
                    <div class="team-select-wrap">
                      <span class="mini-crest" :style="{ '--team-color': getTeam(match.homeTeamId).primary, '--team-accent': getTeam(match.homeTeamId).secondary }" aria-hidden="true">
                        <img :src="teamDataUrls[match.homeTeamId] || getTeam(match.homeTeamId).logo" :alt="getTeam(match.homeTeamId).name" />
                      </span>
                      <select :id="`home-team-${mIdx}`" v-model="match.homeTeamId">
                        <option v-for="team in teams" :key="team.id" :value="team.id" :disabled="team.id === match.awayTeamId">{{ team.fullName }}</option>
                      </select>
                      <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6 8 4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.8" /></svg>
                    </div>
                  </div>
                </div>

                <p v-if="match.awayTeamId === match.homeTeamId" class="field-error">Elige dos equipos diferentes para este juego.</p>
              </div>
            </TransitionGroup>
          </section>

          <!-- SECTION 02: GAME DETAILS (STABLE LAYOUT) -->
          <section class="form-section">
            <div class="section-heading"><span>02</span><div><h2>Datos del juego</h2><p>Define cuándo y dónde</p></div></div>
            <div class="field-grid">
              <label class="input-field"><span>Fecha</span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M5 3v3m10-3v3M3.5 8h13M4 5h12a1 1 0 0 1 1 1v10H3V6a1 1 0 0 1 1-1Z" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" /></svg><input v-model="gameDate" type="date" required /></span></label>
              <label class="input-field"><span>Estadio</span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 17s5-4.6 5-9a5 5 0 1 0-10 0c0 4.4 5 9 5 9Z" fill="none" stroke="currentColor" stroke-width="1.5" /><circle cx="10" cy="8" r="1.8" fill="none" stroke="currentColor" stroke-width="1.5" /></svg><input v-model="venue" list="venues" maxlength="54" placeholder="Nombre del estadio" required /></span></label>
              <datalist id="venues"><option value="Estadio Felipe Rivas" /><option value="Campo Deportivo Principal" /><option value="Estadio Municipal" /></datalist>
              <label class="input-field field-wide"><span>Etiqueta <small>Opcional</small></span><span class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M4 4h8l4 4-8 8-4-4V4Z" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round" /><circle cx="8" cy="8" r="1.2" fill="currentColor" /></svg><input v-model="eyebrow" maxlength="28" placeholder="Temporada regular" /></span></label>
            </div>
          </section>

          <!-- SECTION 03: POSTER BACKGROUND IMAGE -->
          <section class="form-section compact-section">
            <div class="section-heading theme-heading"><span>03</span><div><h2>Fondo del cartel</h2><p>Elige el ambiente del estadio</p></div></div>
            <div class="bg-options" role="radiogroup" aria-label="Fondo del cartel">
              <label v-for="bg in backgrounds" :key="bg.id" :class="{ active: selectedBgId === bg.id }">
                <input v-model="selectedBgId" type="radio" name="posterBg" :value="bg.id" />
                <span class="bg-thumb" :style="{ backgroundImage: `url(${bgDataUrls[bg.id] || bg.asset})` }"></span>
                <span>{{ bg.name }}</span>
                <svg viewBox="0 0 16 16" aria-hidden="true"><path d="m4 8 2.5 2.5L12 5" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
              </label>
            </div>
          </section>

          <!-- SECTION 04: THEME ACCENT COLOR -->
          <section class="form-section compact-section">
            <div class="section-heading theme-heading"><span>04</span><div><h2>Color de acento</h2><p>El toque final del cartel</p></div></div>
            <div class="theme-options" role="radiogroup" aria-label="Color de acento">
              <label v-for="theme in themes" :key="theme.id" :class="{ active: selectedThemeId === theme.id }">
                <input v-model="selectedThemeId" type="radio" name="theme" :value="theme.id" />
                <i :style="{ background: theme.accent }"></i>
                <span>{{ theme.name }}</span>
                <svg viewBox="0 0 16 16" aria-hidden="true"><path d="m4 8 2.5 2.5L12 5" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
              </label>
            </div>
          </section>

          <div class="form-actions">
            <button class="download-button" type="submit" :disabled="isExporting">
              <svg v-if="!isExporting" viewBox="0 0 24 24" aria-hidden="true"><path d="M12 4v11m0 0 4-4m-4 4-4-4M5 19h14" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
              <span v-else class="button-spinner" aria-hidden="true"></span>
              {{ isExporting ? 'Preparando cartel…' : 'Descargar cartel PNG' }}
            </button>
            <p><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M8 1.5 13.5 4v3.8c0 3.3-2.3 5.6-5.5 6.7-3.2-1.1-5.5-3.4-5.5-6.7V4L8 1.5Z" fill="none" stroke="currentColor" stroke-width="1.2" /><path d="m5.5 8 1.6 1.6 3.5-3.5" fill="none" stroke="currentColor" stroke-width="1.2" stroke-linecap="round" /></svg> Se procesa en tu navegador. Nada se publica.</p>
          </div>
        </form>
      </aside>

      <section class="preview-panel" aria-labelledby="preview-title">
        <div class="preview-toolbar"><div><span class="live-dot"></span><div><h2 id="preview-title">Vista previa</h2><p>Se actualiza en tiempo real</p></div></div><span class="format-pill">PUBLICACIÓN · 1:1</span></div>
        <div class="poster-stage">
          <div class="poster-shadow" aria-hidden="true"></div>
          <svg ref="posterSvg" class="poster" viewBox="0 0 1080 1080" role="img" :aria-label="`Cartel de softball, ${formattedDate}, ${venueDisplay}`" xmlns="http://www.w3.org/2000/svg">
            <defs>
              <linearGradient id="posterShade" x1="0" y1="0" x2="0" y2="1"><stop offset="0" :stop-color="selectedTheme.ink" stop-opacity=".76" /><stop offset=".48" stop-color="#07131D" stop-opacity=".68" /><stop offset="1" stop-color="#03090D" stop-opacity=".97" /></linearGradient>
              <linearGradient id="cardFill" x1="0" y1="0" x2="0" y2="1"><stop offset="0" stop-color="#10232C" stop-opacity=".89" /><stop offset="1" stop-color="#071219" stop-opacity=".94" /></linearGradient>
              <radialGradient id="stadiumGlow" cx="50%" cy="35%" r="58%"><stop offset="0" :stop-color="selectedTheme.accent" stop-opacity=".14" /><stop offset="1" stop-color="#000000" stop-opacity="0" /></radialGradient>
              <linearGradient id="accentLine" x1="0" y1="0" x2="1" y2="0"><stop offset="0" :stop-color="selectedTheme.accent" stop-opacity="0" /><stop offset=".5" :stop-color="selectedTheme.accent" /><stop offset="1" :stop-color="selectedTheme.accent" stop-opacity="0" /></linearGradient>
              <pattern id="dotGrid" width="24" height="24" patternUnits="userSpaceOnUse"><circle cx="2" cy="2" r="1.2" fill="#FFFFFF" fill-opacity=".09" /></pattern>
              <filter id="softShadow" x="-30%" y="-30%" width="160%" height="170%"><feDropShadow dx="0" dy="18" stdDeviation="18" flood-color="#000000" flood-opacity=".42" /></filter>
              <filter id="logoShadow" x="-30%" y="-30%" width="160%" height="170%"><feDropShadow dx="0" dy="12" stdDeviation="10" flood-color="#000000" flood-opacity=".5" /></filter>
              <clipPath id="leagueLogoClip"><rect x="0" y="0" width="44" height="44" rx="10" ry="10" /></clipPath>
              <clipPath id="logo-away"><circle cx="0" cy="0" r="91" /></clipPath>
              <clipPath id="logo-home"><circle cx="0" cy="0" r="91" /></clipPath>
              <clipPath id="crestClip2"><circle cx="0" cy="0" r="50" /></clipPath>
              <clipPath id="crestClip3"><circle cx="0" cy="0" r="38" /></clipPath>
              <clipPath id="crestClip4"><circle cx="0" cy="0" r="30" /></clipPath>
            </defs>

            <!-- Background Image and overlays -->
            <rect width="1080" height="1080" fill="#061018" />
            <image :href="bgDataUrls[currentBg.id] || currentBg.asset" width="1080" height="1080" preserveAspectRatio="xMidYMid slice" />
            <rect width="1080" height="1080" fill="url(#posterShade)" /><rect width="1080" height="1080" fill="url(#stadiumGlow)" /><rect width="1080" height="1080" fill="url(#dotGrid)" opacity=".26" />
            <g opacity=".24" fill="none" :stroke="selectedTheme.accent" stroke-width="2"><path d="M540 355 743 558 540 761 337 558Z" /><path d="M540 400 698 558 540 716 382 558Z" stroke-dasharray="7 13" /><circle cx="540" cy="558" r="7" :fill="selectedTheme.accent" stroke="none" /></g>

            <g font-family="Arial, Helvetica, sans-serif">
              <!-- HEADER WITH LEAGUE LOGO (ROUNDED CORNERS) -->
              <g transform="translate(72 54)">
                <rect x="-2" y="-2" width="48" height="48" rx="12" ry="12" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".25" stroke-width="1.5" />
                <rect x="0" y="0" width="44" height="44" rx="10" ry="10" :fill="selectedTheme.accent" />
                <image
                  :href="leagueLogoDataUrl || logoLigaAsset"
                  x="0"
                  y="0"
                  width="44"
                  height="44"
                  preserveAspectRatio="xMidYMid slice"
                  clip-path="url(#leagueLogoClip)"
                />
                <text x="60" y="19" fill="#FFFFFF" font-size="23" font-weight="800" letter-spacing="3">LIGA CRISTIANA</text>
                <text x="60" y="39" fill="#FFFFFF" fill-opacity=".52" font-size="11" font-weight="600" letter-spacing="3.2">JUGANDO PARA CRISTO</text>
              </g>

              <g transform="translate(1008 61)" text-anchor="end"><text y="15" fill="#FFFFFF" fill-opacity=".54" font-size="11" font-weight="700" letter-spacing="2.5">JORNADA DE JUEGO</text><text y="38" fill="#FFFFFF" font-size="18" font-weight="800" letter-spacing="2">{{ gameYear }}</text></g>
              <path d="M72 122H1008" stroke="#FFFFFF" stroke-opacity=".17" /><path d="M72 122H245" :stroke="selectedTheme.accent" stroke-width="3" />
              <text x="540" y="174" text-anchor="middle" :fill="selectedTheme.accentSoft" font-size="15" font-weight="800" letter-spacing="6.2">{{ eyebrowDisplay }}</text>
              <text x="540" y="229" text-anchor="middle" fill="#FFFFFF" font-size="53" font-weight="900" letter-spacing="5">{{ computedPosterTitle }}</text>
              <text x="540" y="267" text-anchor="middle" fill="#FFFFFF" fill-opacity=".68" font-size="17" font-weight="700" letter-spacing="2.7">{{ formattedDate }}</text>

              <!-- ============================================== -->
              <!-- LAYOUT 1: SINGLE MATCH (DETAILED CHAMPIONSHIP) -->
              <!-- ============================================== -->
              <template v-if="matches.length === 1">
                <g filter="url(#softShadow)"><rect x="72" y="302" width="936" height="496" rx="26" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" /><path d="M72 363H1008" stroke="#FFFFFF" stroke-opacity=".12" /><path d="M540 363V770" stroke="#FFFFFF" stroke-opacity=".07" /><path d="M72 302H1008" :stroke="selectedTheme.accent" stroke-width="3" opacity=".9" /></g>

                <!-- TEAMS SHOWCASE -->
                <g v-for="item in posterTeamsSingle" :key="item.slot" :transform="`translate(${item.x} 0)`">
                  <text x="0" y="341" text-anchor="middle" fill="#FFFFFF" fill-opacity=".48" font-size="13" font-weight="800" letter-spacing="4.5">{{ item.label }}</text>
                  
                  <!-- CREST WITH EMBEDDED TEAM LOGO -->
                  <g transform="translate(0 488)" filter="url(#logoShadow)">
                    <circle r="105" fill="#02080C" fill-opacity=".85" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                    <circle r="95" :fill="item.team.primary" />
                    <circle r="88" fill="#FFFFFF" fill-opacity=".08" />
                    <image
                      :href="teamDataUrls[item.team.id] || item.team.logo"
                      x="-76"
                      y="-76"
                      width="152"
                      height="152"
                      preserveAspectRatio="xMidYMid meet"
                      :clip-path="`url(#logo-${item.slot})`"
                    />
                  </g>

                  <!-- OPTIMIZED TEAM NAME -->
                  <text
                    x="0"
                    text-anchor="middle"
                    fill="#FFFFFF"
                    :font-size="formatTeamName(item.team.name).fontSize"
                    font-weight="900"
                    letter-spacing="1"
                  >
                    <tspan
                      v-for="(line, lIdx) in formatTeamName(item.team.name).lines"
                      :key="lIdx"
                      x="0"
                      :y="formatTeamName(item.team.name).startY + lIdx * formatTeamName(item.team.name).lineHeight"
                    >
                      {{ line }}
                    </tspan>
                  </text>

                  <!-- SUBTITLE CITY / LEAGUE -->
                  <text
                    x="0"
                    :y="formatTeamName(item.team.name).cityY"
                    text-anchor="middle"
                    :fill="selectedTheme.accentSoft"
                    font-size="14"
                    font-weight="700"
                    letter-spacing="4.5"
                  >
                    {{ item.team.city.toLocaleUpperCase('es-VE') }}
                  </text>
                </g>

                <g transform="translate(540 500)"><circle r="53" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" /><circle r="43" :fill="selectedTheme.accent" /><path d="M-29-24Q0-4 29-24M-29 24Q0 4 29 24" fill="none" stroke="#07131D" stroke-opacity=".45" stroke-width="2.5" /><text y="7" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="27" font-weight="900" letter-spacing="-1">VS</text></g>
              </template>

              <!-- ============================================== -->
              <!-- LAYOUT 2: TWO MATCHES                          -->
              <!-- ============================================== -->
              <template v-else-if="matches.length === 2">
                <g v-for="(m, mIdx) in matches" :key="m.id" :transform="`translate(72 ${300 + mIdx * 256})`">
                  <g filter="url(#softShadow)">
                    <rect width="936" height="240" rx="22" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" />
                    <rect width="6" height="240" rx="3" :fill="selectedTheme.accent" />
                  </g>

                  <!-- AWAY TEAM (Left) -->
                  <g transform="translate(195 120)">
                    <g transform="translate(-105 0)" filter="url(#logoShadow)">
                      <circle r="56" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                      <circle r="50" :fill="getTeam(m.awayTeamId).primary" />
                      <image
                        :href="teamDataUrls[m.awayTeamId] || getTeam(m.awayTeamId).logo"
                        x="-40"
                        y="-40"
                        width="80"
                        height="80"
                        preserveAspectRatio="xMidYMid meet"
                        clip-path="url(#crestClip2)"
                      />
                    </g>
                    <text
                      x="-32"
                      fill="#FFFFFF"
                      :font-size="formatMultiName(getTeam(m.awayTeamId).name, 'two').fontSize"
                      font-weight="900"
                      letter-spacing=".5"
                    >
                      <tspan
                        v-for="(line, lIdx) in formatMultiName(getTeam(m.awayTeamId).name, 'two').lines"
                        :key="lIdx"
                        x="-32"
                        :y="formatMultiName(getTeam(m.awayTeamId).name, 'two').startY + lIdx * formatMultiName(getTeam(m.awayTeamId).name, 'two').lineHeight"
                      >
                        {{ line }}
                      </tspan>
                    </text>
                    <text x="-32" :y="formatMultiName(getTeam(m.awayTeamId).name, 'two').labelY" :fill="selectedTheme.accentSoft" font-size="12" font-weight="700" letter-spacing="2">VISITANTE</text>
                  </g>

                  <!-- VS Center & Time below -->
                  <g transform="translate(468 96)">
                    <circle r="38" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                    <circle r="30" :fill="selectedTheme.accent" />
                    <text y="5" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="18" font-weight="900">VS</text>
                    <text y="68" text-anchor="middle" fill="#FFFFFF" fill-opacity=".92" font-size="18" font-weight="900" letter-spacing=".5">{{ formatTime(m.time) }}</text>
                  </g>

                  <!-- HOME TEAM (Right) -->
                  <g transform="translate(635 120)">
                    <text
                      x="140"
                      text-anchor="end"
                      fill="#FFFFFF"
                      :font-size="formatMultiName(getTeam(m.homeTeamId).name, 'two').fontSize"
                      font-weight="900"
                      letter-spacing=".5"
                    >
                      <tspan
                        v-for="(line, lIdx) in formatMultiName(getTeam(m.homeTeamId).name, 'two').lines"
                        :key="lIdx"
                        x="140"
                        :y="formatMultiName(getTeam(m.homeTeamId).name, 'two').startY + lIdx * formatMultiName(getTeam(m.homeTeamId).name, 'two').lineHeight"
                      >
                        {{ line }}
                      </tspan>
                    </text>
                    <text x="140" :y="formatMultiName(getTeam(m.homeTeamId).name, 'two').labelY" text-anchor="end" :fill="selectedTheme.accentSoft" font-size="12" font-weight="700" letter-spacing="2">LOCAL</text>
                    <g transform="translate(210 0)" filter="url(#logoShadow)">
                      <circle r="56" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                      <circle r="50" :fill="getTeam(m.homeTeamId).primary" />
                      <image
                        :href="teamDataUrls[m.homeTeamId] || getTeam(m.homeTeamId).logo"
                        x="-40"
                        y="-40"
                        width="80"
                        height="80"
                        preserveAspectRatio="xMidYMid meet"
                        clip-path="url(#crestClip2)"
                      />
                    </g>
                  </g>
                </g>
              </template>

              <!-- ============================================== -->
              <!-- LAYOUT 3: THREE MATCHES                        -->
              <!-- ============================================== -->
              <template v-else-if="matches.length === 3">
                <g v-for="(m, mIdx) in matches" :key="m.id" :transform="`translate(72 ${300 + mIdx * 172})`">
                  <g filter="url(#softShadow)">
                    <rect width="936" height="156" rx="18" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" />
                    <rect width="5" height="156" rx="2.5" :fill="selectedTheme.accent" />
                  </g>

                  <!-- AWAY (Left) -->
                  <g transform="translate(68 78)">
                    <g filter="url(#logoShadow)">
                      <circle r="44" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="39" :fill="getTeam(m.awayTeamId).primary" />
                      <image
                        :href="teamDataUrls[m.awayTeamId] || getTeam(m.awayTeamId).logo"
                        x="-30"
                        y="-30"
                        width="60"
                        height="60"
                        preserveAspectRatio="xMidYMid meet"
                        clip-path="url(#crestClip3)"
                      />
                    </g>
                    <text
                      x="58"
                      fill="#FFFFFF"
                      :font-size="formatMultiName(getTeam(m.awayTeamId).name, 'three').fontSize"
                      font-weight="900"
                      letter-spacing=".4"
                    >
                      <tspan
                        v-for="(line, lIdx) in formatMultiName(getTeam(m.awayTeamId).name, 'three').lines"
                        :key="lIdx"
                        x="58"
                        :y="formatMultiName(getTeam(m.awayTeamId).name, 'three').startY + lIdx * formatMultiName(getTeam(m.awayTeamId).name, 'three').lineHeight"
                      >
                        {{ line }}
                      </tspan>
                    </text>
                  </g>

                  <!-- VS CENTER & TIME DIRECTLY BELOW -->
                  <g transform="translate(468 62)">
                    <circle r="28" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                    <circle r="22" :fill="selectedTheme.accent" />
                    <text y="4" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="14" font-weight="900">VS</text>
                    <text y="52" text-anchor="middle" fill="#FFFFFF" fill-opacity=".92" font-size="16" font-weight="900" letter-spacing=".5">{{ formatTime(m.time) }}</text>
                  </g>

                  <!-- HOME (Right) -->
                  <g transform="translate(868 78)">
                    <text
                      x="-58"
                      text-anchor="end"
                      fill="#FFFFFF"
                      :font-size="formatMultiName(getTeam(m.homeTeamId).name, 'three').fontSize"
                      font-weight="900"
                      letter-spacing=".4"
                    >
                      <tspan
                        v-for="(line, lIdx) in formatMultiName(getTeam(m.homeTeamId).name, 'three').lines"
                        :key="lIdx"
                        x="-58"
                        :y="formatMultiName(getTeam(m.homeTeamId).name, 'three').startY + lIdx * formatMultiName(getTeam(m.homeTeamId).name, 'three').lineHeight"
                      >
                        {{ line }}
                      </tspan>
                    </text>
                    <g filter="url(#logoShadow)">
                      <circle r="44" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="39" :fill="getTeam(m.homeTeamId).primary" />
                      <image
                        :href="teamDataUrls[m.homeTeamId] || getTeam(m.homeTeamId).logo"
                        x="-30"
                        y="-30"
                        width="60"
                        height="60"
                        preserveAspectRatio="xMidYMid meet"
                        clip-path="url(#crestClip3)"
                      />
                    </g>
                  </g>
                </g>
              </template>

              <!-- ============================================== -->
              <!-- LAYOUT 4: FOUR MATCHES                         -->
              <!-- ============================================== -->
              <template v-else>
                <g v-for="(m, mIdx) in matches" :key="m.id" :transform="`translate(72 ${300 + mIdx * 128})`">
                  <g filter="url(#softShadow)">
                    <rect width="936" height="114" rx="16" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" />
                    <rect width="5" height="114" rx="2.5" :fill="selectedTheme.accent" />
                  </g>

                  <!-- AWAY (Left) -->
                  <g transform="translate(68 57)">
                    <g filter="url(#logoShadow)">
                      <circle r="34" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="30" :fill="getTeam(m.awayTeamId).primary" />
                      <image
                        :href="teamDataUrls[m.awayTeamId] || getTeam(m.awayTeamId).logo"
                        x="-24"
                        y="-24"
                        width="48"
                        height="48"
                        preserveAspectRatio="xMidYMid meet"
                        clip-path="url(#crestClip4)"
                      />
                    </g>
                    <text
                      x="48"
                      fill="#FFFFFF"
                      :font-size="formatMultiName(getTeam(m.awayTeamId).name, 'four').fontSize"
                      font-weight="900"
                      letter-spacing=".3"
                    >
                      <tspan
                        v-for="(line, lIdx) in formatMultiName(getTeam(m.awayTeamId).name, 'four').lines"
                        :key="lIdx"
                        x="48"
                        :y="formatMultiName(getTeam(m.awayTeamId).name, 'four').startY + lIdx * formatMultiName(getTeam(m.awayTeamId).name, 'four').lineHeight"
                      >
                        {{ line }}
                      </tspan>
                    </text>
                  </g>

                  <!-- VS CENTER & TIME DIRECTLY BELOW -->
                  <g transform="translate(468 44)">
                    <circle r="22" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.8" />
                    <circle r="17" :fill="selectedTheme.accent" />
                    <text y="4" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="11" font-weight="900">VS</text>
                    <text y="42" text-anchor="middle" fill="#FFFFFF" fill-opacity=".92" font-size="14" font-weight="900" letter-spacing=".5">{{ formatTime(m.time) }}</text>
                  </g>

                  <!-- HOME (Right) -->
                  <g transform="translate(868 57)">
                    <text
                      x="-48"
                      text-anchor="end"
                      fill="#FFFFFF"
                      :font-size="formatMultiName(getTeam(m.homeTeamId).name, 'four').fontSize"
                      font-weight="900"
                      letter-spacing=".3"
                    >
                      <tspan
                        v-for="(line, lIdx) in formatMultiName(getTeam(m.homeTeamId).name, 'four').lines"
                        :key="lIdx"
                        x="-48"
                        :y="formatMultiName(getTeam(m.homeTeamId).name, 'four').startY + lIdx * formatMultiName(getTeam(m.homeTeamId).name, 'four').lineHeight"
                      >
                        {{ line }}
                      </tspan>
                    </text>
                    <g filter="url(#logoShadow)">
                      <circle r="34" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="30" :fill="getTeam(m.homeTeamId).primary" />
                      <image
                        :href="teamDataUrls[m.homeTeamId] || getTeam(m.homeTeamId).logo"
                        x="-24"
                        y="-24"
                        width="48"
                        height="48"
                        preserveAspectRatio="xMidYMid meet"
                        clip-path="url(#crestClip4)"
                      />
                    </g>
                  </g>
                </g>
              </template>

              <!-- BOTTOM VENUE CARD -->
              <g transform="translate(72 844)"><rect width="936" height="142" rx="22" fill="#050D12" fill-opacity=".9" stroke="#FFFFFF" stroke-opacity=".13" /><rect width="8" height="142" rx="4" :fill="selectedTheme.accent" />
                <g transform="translate(47 34)"><path d="M18 0c-9.5 0-17 7.4-17 17 0 14.2 17 27 17 27s17-12.8 17-27C35 7.4 27.5 0 18 0Zm0 23.2a6.5 6.5 0 1 1 0-13 6.5 6.5 0 0 1 0 13Z" :fill="selectedTheme.accent" /><text x="55" y="10" fill="#FFFFFF" fill-opacity=".46" font-size="12" font-weight="800" letter-spacing="3.2">SEDE</text><text x="55" fill="#FFFFFF" :font-size="venueFontSize" font-weight="900" letter-spacing=".3"><tspan v-for="(line, index) in venueLines" :key="line" x="55" :y="venueLines.length > 1 ? 37 + index * 28 : 47">{{ line }}</tspan></text></g>
                <path d="M720 24V118" stroke="#FFFFFF" stroke-opacity=".13" />
                <g transform="translate(828 25)" text-anchor="middle">
                  <circle cy="17" r="17" fill="none" :stroke="selectedTheme.accent" stroke-width="2.5" />
                  <path d="M0 7v11l7 4" fill="none" :stroke="selectedTheme.accent" stroke-width="2.5" stroke-linecap="round" />
                  <text y="65" fill="#FFFFFF" fill-opacity=".46" font-size="10" font-weight="800" letter-spacing="2">
                    {{ matches.length === 1 ? 'PRIMER PITCHEO' : 'JORNADA' }}
                  </text>
                  <text y="104" fill="#FFFFFF" font-size="29" font-weight="900" letter-spacing=".2">
                    {{ matches.length === 1 ? formatTime(matches[0].time) : `${matches.length} JUEGOS` }}
                  </text>
                </g>
              </g>

              <g transform="translate(72 1030)"><text fill="#FFFFFF" fill-opacity=".42" font-size="11" font-weight="700" letter-spacing="2.4">SOFTBALL · PASIÓN · TRADICIÓN</text><text x="936" text-anchor="end" fill="#FFFFFF" fill-opacity=".42" font-size="11" font-weight="700" letter-spacing="2.4">#DÍADEJUEGO</text></g>
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
