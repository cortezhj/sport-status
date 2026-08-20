<script setup lang="ts">
import { computed, nextTick, onMounted, reactive, ref } from 'vue'
import stadiumAsset from './assets/stadium-night.jpg'
import bgStadiumNight from './assets/backgrounds/stadium-night.jpg'
import bgSoftballDay from './assets/backgrounds/softball-day.jpg'
import bgStadiumLights from './assets/backgrounds/stadium-lights.jpg'
import bgStadiumSunset from './assets/backgrounds/stadium-sunset.jpg'
import bgResultsNight from './assets/backgrounds/results-night.jpg'
import bgResultsElectric from './assets/backgrounds/results-electric.jpg'
import bgResultsSunset from './assets/backgrounds/results-sunset.jpg'
import bgResultsChampions from './assets/backgrounds/results-champions.jpg'
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
type ViewMode = 'schedule' | 'results'

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

interface ResultScore {
  runs: number
  hits: number
  errors: number
}

interface ResultMatch {
  id: string
  awayTeamId: string
  homeTeamId: string
  awayScore: ResultScore
  homeScore: ResultScore
  status: string
  note?: string
  showNote?: boolean
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

const resultsThemes: PosterTheme[] = [
  { id: 'res-gold', name: 'Dorado', accent: '#FBBF24', accentSoft: '#FEF08A', ink: '#050E18' },
  { id: 'res-emerald', name: 'Esmeralda', accent: '#10B981', accentSoft: '#A7F3D0', ink: '#041610' },
  { id: 'res-cyan', name: 'Cyan', accent: '#06B6D4', accentSoft: '#A5F3FC', ink: '#02141F' },
]

const resultsBackgrounds: PosterBg[] = [
  { id: 'res-night', name: 'Noche Épica', asset: bgResultsNight },
  { id: 'res-electric', name: 'Crepúsculo Pro', asset: bgResultsElectric },
  { id: 'res-sunset', name: 'Atardecer Dorado', asset: bgResultsSunset },
  { id: 'res-champions', name: 'Arena Champions', asset: bgResultsChampions },
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

// Active mode: 'schedule' or 'results'
const currentView = ref<ViewMode>('schedule')

const posterSvg = ref<SVGSVGElement | null>(null)
const stadiumDataUrl = ref('')
const leagueLogoDataUrl = ref('')
const teamDataUrls = reactive<Record<string, string>>({})
const bgDataUrls = reactive<Record<string, string>>({})

const isExporting = ref(false)
const notice = ref('')
const noticeTimer = ref<number | undefined>()

// ----------------------------------------------------
// SCHEDULE STATE
// ----------------------------------------------------
const matches = reactive<Match[]>([
  { id: 'm1', awayTeamId: 'cristo-el-salvador', homeTeamId: 'dios-es-bueno', time: '12:00' },
])

const gameDate = ref(nextSaturday())
const venue = ref('Estadio Felipe Rivas')
const eyebrow = ref('TEMPORADA REGULAR')
const posterTitle = ref('')
const selectedThemeId = ref('lights')
const selectedBgId = ref('night')

// ----------------------------------------------------
// RESULTS STATE
// ----------------------------------------------------
const resultMatches = reactive<ResultMatch[]>([
  {
    id: 'rm1',
    awayTeamId: 'cristo-el-salvador',
    homeTeamId: 'dios-es-bueno',
    awayScore: { runs: 0, hits: 0, errors: 0 },
    homeScore: { runs: 0, hits: 0, errors: 0 },
    status: 'FINAL',
    note: '',
    showNote: false,
  },
])

const resultsDate = ref(nextSaturday())
const resultsEyebrow = ref('TEMPORADA REGULAR')
const resultsPosterTitle = ref('')
const resultsThemeId = ref('res-gold')
const resultsBgId = ref('res-night')

const getTeam = (id: string): Team => {
  return teams.find((team) => team.id === id) ?? teams[0]
}

// ----------------------------------------------------
// SCHEDULE COMPUTED
// ----------------------------------------------------
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
  const hour = parseInt(hourValue, 10)
  if (Number.isNaN(hour)) return timeStr
  const suffix = hour >= 12 ? 'PM' : 'AM'
  const hour12 = hour % 12 === 0 ? 12 : hour % 12
  return `${hour12}:${minutes.padStart(2, '0')} ${suffix}`
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

// ----------------------------------------------------
// RESULTS COMPUTED
// ----------------------------------------------------
const selectedResultsTheme = computed(() => resultsThemes.find((theme) => theme.id === resultsThemeId.value) ?? resultsThemes[0])
const currentResultsBg = computed(() => resultsBackgrounds.find((b) => b.id === resultsBgId.value) ?? resultsBackgrounds[0])

const hasAnyResultConflict = computed(() => {
  return resultMatches.some((m) => m.awayTeamId === m.homeTeamId)
})

const isResultsFormComplete = computed(() => {
  return Boolean(
    resultsDate.value &&
    !hasAnyResultConflict.value &&
    resultMatches.length > 0 &&
    resultMatches.every((m) => typeof m.awayScore.runs === 'number' && typeof m.homeScore.runs === 'number')
  )
})

// Predetermined title: "RESULTADOS DE JORNADA"
const computedResultsTitle = computed(() => {
  if (resultsPosterTitle.value.trim()) return resultsPosterTitle.value.trim().toLocaleUpperCase('es-VE')
  return 'RESULTADOS DE JORNADA'
})

const formattedResultsDate = computed(() => {
  if (!resultsDate.value) return 'FECHA POR CONFIRMAR'
  const date = new Date(`${resultsDate.value}T12:00:00`)
  return new Intl.DateTimeFormat('es-VE', { weekday: 'long', day: '2-digit', month: 'long' })
    .format(date)
    .replace(',', ' ·')
    .toLocaleUpperCase('es-VE')
})

const resultsGameYear = computed(() => resultsDate.value ? new Date(`${resultsDate.value}T12:00:00`).getFullYear() : new Date().getFullYear())
const resultsEyebrowDisplay = computed(() => resultsEyebrow.value.trim().toLocaleUpperCase('es-VE').slice(0, 28) || 'TEMPORADA REGULAR')

// Format team name dynamically for best typography in the SVG poster
const formatTeamName = (name: string) => {
  const upper = name.toLocaleUpperCase('es-VE').trim()
  if (upper === 'FRATERNIDAD CRISTIANA') {
    return { lines: ['FRATERNIDAD', 'CRISTIANA'], fontSize: 27, startY: 642, lineHeight: 31, cityY: 704 }
  }
  if (upper === 'CRISTO EL SALVADOR') {
    return { lines: ['CRISTO EL', 'SALVADOR'], fontSize: 28, startY: 642, lineHeight: 31, cityY: 704 }
  }
  if (upper === 'DIOS ES BUENO') {
    return { lines: ['DIOS ES', 'BUENO'], fontSize: 30, startY: 642, lineHeight: 32, cityY: 704 }
  }
  if (upper === 'LUZ Y VIDA') {
    return { lines: ['LUZ Y VIDA'], fontSize: 36, startY: 658, lineHeight: 0, cityY: 690 }
  }
  if (upper.length <= 10) {
    return { lines: [upper], fontSize: upper.length <= 4 ? 42 : 38, startY: 658, lineHeight: 0, cityY: 690 }
  }
  const words = upper.split(' ')
  if (words.length >= 2) {
    const mid = Math.ceil(words.length / 2)
    return { lines: [words.slice(0, mid).join(' '), words.slice(mid).join(' ')], fontSize: 28, startY: 642, lineHeight: 31, cityY: 704 }
  }
  return { lines: [upper], fontSize: 32, startY: 658, lineHeight: 0, cityY: 690 }
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

// ----------------------------------------------------
// SCHEDULE ACTIONS
// ----------------------------------------------------
const addMatch = () => {
  if (matches.length >= 4) {
    showNotice('Máximo 4 encuentros por cartel diario.')
    return
  }
  const defaultTimes = ['12:00', '13:30', '15:00', '16:30']
  const nextTime = defaultTimes[matches.length] || '16:30'
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

// ----------------------------------------------------
// RESULTS ACTIONS
// ----------------------------------------------------
const addResultMatch = () => {
  if (resultMatches.length >= 4) {
    showNotice('Máximo 4 resultados por cartel.')
    return
  }
  const usedTeams = new Set(resultMatches.flatMap((m) => [m.awayTeamId, m.homeTeamId]))
  const available = teams.filter((t) => !usedTeams.has(t.id))
  const awayId = available[0]?.id || teams[(resultMatches.length * 2) % teams.length].id
  const homeId = available[1]?.id || teams[(resultMatches.length * 2 + 1) % teams.length].id

  resultMatches.push({
    id: `rm_${Date.now()}_${Math.random().toString(36).slice(2, 6)}`,
    awayTeamId: awayId,
    homeTeamId: homeId,
    awayScore: { runs: 0, hits: 0, errors: 0 },
    homeScore: { runs: 0, hits: 0, errors: 0 },
    status: 'FINAL',
    note: '',
    showNote: false,
  })
  showNotice(`Resultado #${resultMatches.length} añadido.`)
}

const removeResultMatch = (index: number) => {
  if (resultMatches.length <= 1) {
    showNotice('Debe haber al menos un resultado.')
    return
  }
  resultMatches.splice(index, 1)
  showNotice('Resultado eliminado.')
}

const toggleNote = (match: ResultMatch) => {
  match.showNote = !match.showNote
  if (!match.showNote) {
    match.note = ''
  }
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
    for (const bg of resultsBackgrounds) {
      bgDataUrls[bg.id] = await urlToDataUrl(bg.asset)
    }
  } catch (e) {
    console.warn('Error precargando imágenes:', e)
  }
}

// ----------------------------------------------------
// LOCALSTORAGE PERSISTENCE & AUTO-COMPLETE
// ----------------------------------------------------
const STORAGE_KEY_SCHEDULE = 'sport_status_saved_schedule'
const STORAGE_KEY_RESULTS = 'sport_status_saved_results'

const saveScheduleToStorage = () => {
  try {
    const payload = {
      matches: matches.map((m) => ({ ...m })),
      gameDate: gameDate.value,
      venue: venue.value,
      eyebrow: eyebrow.value,
      posterTitle: posterTitle.value,
      selectedThemeId: selectedThemeId.value,
      selectedBgId: selectedBgId.value,
      savedAt: new Date().toISOString(),
    }
    localStorage.setItem(STORAGE_KEY_SCHEDULE, JSON.stringify(payload))
    return payload
  } catch (e) {
    console.warn('No se pudo guardar la cartelera en localStorage:', e)
    return null
  }
}

const syncResultsWithSchedule = (
  sourceMatches: Match[],
  sourceDate?: string,
  sourceEyebrow?: string,
  forceResetScores = false
) => {
  if (sourceDate) resultsDate.value = sourceDate
  if (sourceEyebrow !== undefined) resultsEyebrow.value = sourceEyebrow

  const updatedResultMatches: ResultMatch[] = sourceMatches.map((m, idx) => {
    const existing = resultMatches[idx]
    const isSameMatch =
      !forceResetScores &&
      existing &&
      existing.awayTeamId === m.awayTeamId &&
      existing.homeTeamId === m.homeTeamId

    return {
      id: `rm_${idx + 1}_${Date.now()}_${Math.random().toString(36).slice(2, 5)}`,
      awayTeamId: m.awayTeamId,
      homeTeamId: m.homeTeamId,
      awayScore: isSameMatch ? { ...existing.awayScore } : { runs: 0, hits: 0, errors: 0 },
      homeScore: isSameMatch ? { ...existing.homeScore } : { runs: 0, hits: 0, errors: 0 },
      status: isSameMatch && existing.status ? existing.status : 'FINAL',
      note: isSameMatch ? (existing.note || '') : '',
      showNote: isSameMatch ? Boolean(existing.showNote || (existing.note && existing.note.trim())) : false,
    }
  })

  resultMatches.splice(0, resultMatches.length, ...updatedResultMatches)
  saveResultsToStorage()
}

const saveResultsToStorage = () => {
  try {
    const payload = {
      resultMatches: resultMatches.map((rm) => ({
        ...rm,
        awayScore: { ...rm.awayScore },
        homeScore: { ...rm.homeScore },
      })),
      resultsDate: resultsDate.value,
      resultsEyebrow: resultsEyebrow.value,
      resultsPosterTitle: resultsPosterTitle.value,
      resultsThemeId: resultsThemeId.value,
      resultsBgId: resultsBgId.value,
      savedAt: new Date().toISOString(),
    }
    localStorage.setItem(STORAGE_KEY_RESULTS, JSON.stringify(payload))
  } catch (e) {
    console.warn('No se pudo guardar resultados en localStorage:', e)
  }
}

const loadMatchesFromSchedule = () => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY_SCHEDULE)
    if (raw) {
      const data = JSON.parse(raw)
      if (data && Array.isArray(data.matches) && data.matches.length > 0) {
        syncResultsWithSchedule(data.matches, data.gameDate, data.eyebrow, false)
        showNotice('¡Juegos cargados desde la última jornada guardada!')
        return
      }
    }
  } catch (e) {
    console.warn('Error leyendo schedule de localStorage:', e)
  }

  // Fallback to in-memory matches if not saved yet
  if (matches.length > 0) {
    syncResultsWithSchedule(matches, gameDate.value, eyebrow.value, false)
    showNotice('¡Juegos cargados desde la cartelera actual!')
  } else {
    showNotice('No hay juegos guardados para cargar.')
  }
}

const downloadPoster = async () => {
  const isResults = currentView.value === 'results'
  const isComplete = isResults ? isResultsFormComplete.value : isFormComplete.value
  const hasConflict = isResults ? hasAnyResultConflict.value : hasAnyConflict.value

  if (!isComplete || !posterSvg.value) {
    showNotice(hasConflict ? 'Elige equipos diferentes en cada juego.' : 'Completa los datos del formulario.')
    return
  }
  isExporting.value = true
  let svgUrl = ''
  try {
    await preloadAssets()
    if (document.fonts) {
      await document.fonts.ready
    }
    await nextTick()

    const svgClone = posterSvg.value.cloneNode(true) as SVGSVGElement
    svgClone.setAttribute('xmlns', 'http://www.w3.org/2000/svg')
    svgClone.setAttribute('width', '1080')
    svgClone.setAttribute('height', '1080')

    // 💡 SOLUCIÓN: Inyectar estilos y fuentes web directamente dentro del SVG
    const styleElement = document.createElementNS('http://www.w3.org/2000/svg', 'style')
    
    // Si usas Google Fonts, importa aquí la URL exacta de tu tipografía:
    styleElement.textContent = `
      @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;800;900&display=swap');
      * {
        font-family: 'Montserrat', sans-serif !important;
      }
    `
    svgClone.insertBefore(styleElement, svgClone.firstChild)

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
    const targetDate = isResults ? resultsDate.value : gameDate.value
    let dateSuffix = 'fecha'
    if (targetDate) {
      const date = new Date(`${targetDate}T12:00:00`)
      const weekday = new Intl.DateTimeFormat('es-VE', { weekday: 'long' }).format(date)
      const cleanWeekday = weekday.normalize('NFD').replace(/[\u0300-\u036f]/g, '').toLowerCase()
      const day = `${date.getDate()}`.padStart(2, '0')
      dateSuffix = `${cleanWeekday}-${day}`
    }
    const prefix = isResults ? 'softball-liga-cristiana-resultados' : 'softball-liga-cristiana-juegos'
    link.download = `${prefix}-${dateSuffix}.png`
    document.body.appendChild(link)
    link.click()
    link.remove()
    window.setTimeout(() => URL.revokeObjectURL(pngUrl), 1000)

    if (!isResults) {
      saveScheduleToStorage()
      showNotice('Cartel descargado y jornada guardada.')
    } else {
      saveResultsToStorage()
      showNotice('Pizarra descargada en alta resolución.')
    }
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

      <!-- COMPACT MODE SWITCHER TABS -->
      <nav class="mode-tabs" aria-label="Selector de vista">
        <button
          type="button"
          class="mode-tab-btn"
          :class="{ active: currentView === 'schedule' }"
          @click="currentView = 'schedule'"
        >
          <svg viewBox="0 0 20 20" aria-hidden="true"><path d="M6 2v3M14 2v3M3.5 8h13M5 4h10a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2z" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
          <span>Juegos</span>
        </button>
        <button
          type="button"
          class="mode-tab-btn"
          :class="{ active: currentView === 'results' }"
          @click="currentView = 'results'"
        >
          <svg viewBox="0 0 20 20" aria-hidden="true"><path d="M6 3h8v3a4 4 0 0 1-8 0V3zM4 5H2a1 1 0 0 0-1 1v1a3 3 0 0 0 3 3h2M16 5h2a1 1 0 0 1 1 1v1a3 3 0 0 1-3 3h-2M10 10v4M7 17h6M10 14h-2a2 2 0 0 0-2 2v1h8v-1a2 2 0 0 0-2-2h-2z" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
          <span>Resultados</span>
        </button>
      </nav>

      <div class="topbar-meta">
        <span class="autosave" :class="{ invalid: currentView === 'schedule' ? !isFormComplete : !isResultsFormComplete }">
          <i></i> {{ (currentView === 'schedule' ? isFormComplete : isResultsFormComplete) ? 'Diseño listo' : 'Completa los datos' }}
        </span>
        <span class="resolution">1080 × 1080 px</span>
      </div>
    </header>

    <main class="workspace">
      <aside class="control-panel">
        <!-- ============================================== -->
        <!-- VIEW 1: SCHEDULE CONTROL PANEL                 -->
        <!-- ============================================== -->
        <template v-if="currentView === 'schedule'">
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
                  <!-- TOP HEADER BAR -->
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

            <!-- SECTION 02: GAME DETAILS -->
            <section class="form-section">
              <div class="section-heading"><span>02</span><div><h2>Datos del juego</h2><p>Define cuándo y dónde</p></div></div>
              <div class="field-grid">
                <label class="input-field"><span>Fecha</span><div class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M6 2v3M14 2v3M3.5 8h13M5 4h10a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2z" fill="none" stroke="currentColor" stroke-width="1.6" /></svg><input v-model="gameDate" type="date" required /></div></label>
                <label class="input-field"><span>Sede</span><div class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 2a6 6 0 0 0-6 6c0 4.2 6 10 6 10s6-5.8 6-10a6 6 0 0 0-6-6Zm0 8a2 2 0 1 1 0-4 2 2 0 0 1 0 4Z" fill="none" stroke="currentColor" stroke-width="1.6" /></svg><input v-model="venue" type="text" placeholder="Ej. Estadio Felipe Rivas" required /></div></label>
                <label class="input-field field-wide"><span>Subtítulo <small>Opcional</small></span><div class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M4 6h12M4 10h8M4 14h10" fill="none" stroke="currentColor" stroke-width="1.6" /></svg><input v-model="eyebrow" type="text" placeholder="TEMPORADA REGULAR" /></div></label>
              </div>
            </section>

            <!-- SECTION 03: BACKGROUNDS & THEMES -->
            <section class="form-section compact-section">
              <div class="section-heading theme-heading"><span>03</span><div><h2>Fondo y Estilo</h2><p>Personaliza el diseño visual</p></div></div>
              <div class="bg-options">
                <label v-for="bg in backgrounds" :key="bg.id" :class="{ active: selectedBgId === bg.id }">
                  <input v-model="selectedBgId" type="radio" :value="bg.id" name="background" />
                  <span class="bg-thumb" :style="{ backgroundImage: `url(${bg.asset})` }"></span>
                  <span>{{ bg.name }}</span>
                  <svg viewBox="0 0 16 16" aria-hidden="true"><path d="m3 8 3.5 3.5L13 5" fill="none" stroke="currentColor" stroke-width="2" /></svg>
                </label>
              </div>
              <div class="theme-options" style="margin-top: 10px;">
                <label v-for="theme in themes" :key="theme.id" :class="{ active: selectedThemeId === theme.id }">
                  <input v-model="selectedThemeId" type="radio" :value="theme.id" name="theme" />
                  <i :style="{ background: theme.accent }"></i>
                  <span>{{ theme.name }}</span>
                  <svg viewBox="0 0 16 16" aria-hidden="true"><path d="m3 8 3.5 3.5L13 5" fill="none" stroke="currentColor" stroke-width="2" /></svg>
                </label>
              </div>
            </section>

            <!-- ACTIONS -->
            <div class="form-actions">
              <button class="download-button" type="submit" :disabled="!isFormComplete || isExporting">
                <span v-if="isExporting" class="button-spinner" aria-hidden="true"></span>
                <svg v-else viewBox="0 0 20 20" aria-hidden="true"><path d="M10 3v9m0 0 3.5-3.5M10 12 6.5 8.5M3 14.5v1a1.5 1.5 0 0 0 1.5 1.5h11a1.5 1.5 0 0 0 1.5-1.5v-1" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
                <span>{{ isExporting ? 'Generando pieza...' : 'Descargar Cartel HD (PNG)' }}</span>
              </button>
              <p><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M8 2a6 6 0 1 0 0 12A6 6 0 0 0 8 2Zm0 3v3.5l2.5 1.5" fill="none" stroke="currentColor" stroke-width="1.4" /></svg>Formato optimizado para WhatsApp e Instagram (1080 × 1080 px)</p>
            </div>
          </form>
        </template>

        <!-- ============================================== -->
        <!-- VIEW 2: RESULTS CONTROL PANEL                  -->
        <!-- ============================================== -->
        <template v-else>
          <div class="panel-intro">
            <span class="section-kicker">PIZARRA DE RESULTADOS</span>
            <h1>Marcador del <em>juego.</em></h1>
            <p>Ingresa las carreras, hits y errores de los encuentros finalizados.</p>
          </div>

          <form class="match-form" @submit.prevent="downloadPoster">
            <!-- SECTION 01: RESULTS MATCHES -->
            <section class="form-section">
              <div class="section-heading-row">
                <div class="section-heading">
                  <span>01</span>
                  <div>
                    <h2>{{ resultMatches.length === 1 ? 'El resultado' : 'Los resultados' }}</h2>
                    <p>{{ resultMatches.length === 1 ? 'Marcador y estadísticas R·H·E' : `${resultMatches.length} marcadores registrados (máx. 4)` }}</p>
                  </div>
                </div>
                <div class="section-heading-actions">
                  <button
                    type="button"
                    class="sync-schedule-btn"
                    @click="loadMatchesFromSchedule"
                    title="Cargar y sincronizar los equipos de la cartelera de juegos"
                  >
                    <svg viewBox="0 0 20 20" aria-hidden="true">
                      <path d="M4 4v5h5M16 16v-5h-5M15.5 8.5A6.5 6.5 0 0 0 5.2 6.2L4 9m12 2-1.2 2.8A6.5 6.5 0 0 1 4.5 11.5" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
                    </svg>
                    <span>Jornada</span>
                  </button>
                  <button
                    v-if="resultMatches.length < 4"
                    type="button"
                    class="add-match-btn"
                    @click="addResultMatch"
                    title="Añadir otro resultado a la jornada"
                  >
                    <svg viewBox="0 0 20 20" aria-hidden="true"><path d="M10 4v12m-6-6h12" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" /></svg>
                    <span>Añadir</span>
                  </button>
                </div>
              </div>

              <!-- RESULTS STACK -->
              <TransitionGroup name="match-card" tag="div" class="matches-stack">
                <div
                  v-for="(rMatch, rmIdx) in resultMatches"
                  :key="rMatch.id"
                  class="match-card-box result-card-box"
                >
                  <!-- TOP BAR WITH + NOTA BUTTON IN TOP RIGHT -->
                  <div class="match-box-top">
                    <div class="match-box-top-left">
                      <span class="match-badge-tag">Juego {{ rmIdx + 1 }}</span>
                      <label class="match-status-wrap">
                        <span>Estado:</span>
                        <input v-model="rMatch.status" type="text" placeholder="FINAL" class="status-input" />
                      </label>
                    </div>
                    <div class="match-box-top-actions">
                      <button
                        type="button"
                        class="toggle-note-btn"
                        :class="{ active: rMatch.showNote || (rMatch.note && rMatch.note.trim()) }"
                        @click="toggleNote(rMatch)"
                        title="Agregar o quitar nota para este juego"
                      >
                        <svg v-if="!(rMatch.showNote || (rMatch.note && rMatch.note.trim()))" viewBox="0 0 20 20" aria-hidden="true"><path d="M10 4v12m-6-6h12" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round" /></svg>
                        <svg v-else viewBox="0 0 20 20" aria-hidden="true"><path d="m5 5 10 10m0-10L5 15" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" /></svg>
                        <span>{{ (rMatch.showNote || (rMatch.note && rMatch.note.trim())) ? 'Ocultar' : 'NOTA' }}</span>
                      </button>
                      <button
                        v-if="resultMatches.length > 1"
                        type="button"
                        class="remove-match-btn"
                        @click="removeResultMatch(rmIdx)"
                        title="Eliminar este resultado"
                        aria-label="Eliminar este resultado"
                      >
                        <svg viewBox="0 0 24 24" aria-hidden="true">
                          <path d="M4 7h16M10 11v6M14 11v6M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2l1-12M9 7V4a1 1 0 0 1 1-1h4a1 1 0 0 1 1 1v3" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" />
                        </svg>
                      </button>
                    </div>
                  </div>

                  <!-- TEAMS AND BOX SCORE INPUTS (CLEAN TEAM 1 & TEAM 2) -->
                  <div class="result-teams-grid">
                    <!-- TEAM 1 ROW -->
                    <div class="result-team-row" :class="{ 'is-winner': rMatch.awayScore.runs > rMatch.homeScore.runs }">
                      <div class="team-field-main">
                        <label :for="`res-away-${rmIdx}`">Equipo 1 <span v-if="rMatch.awayScore.runs > rMatch.homeScore.runs" class="winner-pill">Ganador</span></label>
                        <div class="team-select-wrap">
                          <span class="mini-crest" :style="{ '--team-color': getTeam(rMatch.awayTeamId).primary, '--team-accent': getTeam(rMatch.awayTeamId).secondary }" aria-hidden="true">
                            <img :src="teamDataUrls[rMatch.awayTeamId] || getTeam(rMatch.awayTeamId).logo" :alt="getTeam(rMatch.awayTeamId).name" />
                          </span>
                          <select :id="`res-away-${rmIdx}`" v-model="rMatch.awayTeamId">
                            <option v-for="team in teams" :key="team.id" :value="team.id" :disabled="team.id === rMatch.homeTeamId">{{ team.fullName }}</option>
                          </select>
                          <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6 8 4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.8" /></svg>
                        </div>
                      </div>

                      <div class="che-inputs-wrap">
                        <label class="che-field che-runs">
                          <span>R (Runs)</span>
                          <input v-model.number="rMatch.awayScore.runs" type="number" min="0" max="99" required />
                        </label>
                        <label class="che-field">
                          <span>H (Hits)</span>
                          <input v-model.number="rMatch.awayScore.hits" type="number" min="0" max="99" />
                        </label>
                        <label class="che-field">
                          <span>E (Errors)</span>
                          <input v-model.number="rMatch.awayScore.errors" type="number" min="0" max="99" />
                        </label>
                      </div>
                    </div>

                    <!-- TEAM 2 ROW -->
                    <div class="result-team-row" :class="{ 'is-winner': rMatch.homeScore.runs > rMatch.awayScore.runs }">
                      <div class="team-field-main">
                        <label :for="`res-home-${rmIdx}`">Equipo 2 <span v-if="rMatch.homeScore.runs > rMatch.awayScore.runs" class="winner-pill">Ganador</span></label>
                        <div class="team-select-wrap">
                          <span class="mini-crest" :style="{ '--team-color': getTeam(rMatch.homeTeamId).primary, '--team-accent': getTeam(rMatch.homeTeamId).secondary }" aria-hidden="true">
                            <img :src="teamDataUrls[rMatch.homeTeamId] || getTeam(rMatch.homeTeamId).logo" :alt="getTeam(rMatch.homeTeamId).name" />
                          </span>
                          <select :id="`res-home-${rmIdx}`" v-model="rMatch.homeTeamId">
                            <option v-for="team in teams" :key="team.id" :value="team.id" :disabled="team.id === rMatch.awayTeamId">{{ team.fullName }}</option>
                          </select>
                          <svg viewBox="0 0 20 20" aria-hidden="true"><path d="m6 8 4 4 4-4" fill="none" stroke="currentColor" stroke-width="1.8" /></svg>
                        </div>
                      </div>

                      <div class="che-inputs-wrap">
                        <label class="che-field che-runs">
                          <span>R (Runs)</span>
                          <input v-model.number="rMatch.homeScore.runs" type="number" min="0" max="99" required />
                        </label>
                        <label class="che-field">
                          <span>H (Hits)</span>
                          <input v-model.number="rMatch.homeScore.hits" type="number" min="0" max="99" />
                        </label>
                        <label class="che-field">
                          <span>E (Errors)</span>
                          <input v-model.number="rMatch.homeScore.errors" type="number" min="0" max="99" />
                        </label>
                      </div>
                    </div>
                  </div>

                  <!-- NOTE INPUT (REVEALED ON DEMAND) -->
                  <Transition name="note-slide">
                    <div v-if="rMatch.showNote || (rMatch.note && rMatch.note.trim())" class="result-note-wrap">
                      <label :for="`res-note-${rmIdx}`">
                        <span>Nota / Observación:</span>
                        <div class="note-input-shell">
                          <svg viewBox="0 0 20 20" aria-hidden="true"><path d="M4 4h12v12H4zM4 9h12M9 4v12" fill="none" stroke="currentColor" stroke-width="1.5" /></svg>
                          <input
                            :id="`res-note-${rmIdx}`"
                            v-model="rMatch.note"
                            type="text"
                            placeholder="Ej. Victoria por forfeit / Fuera de roster"
                            autofocus
                          />
                        </div>
                      </label>
                    </div>
                  </Transition>

                  <p v-if="rMatch.awayTeamId === rMatch.homeTeamId" class="field-error">Elige dos equipos diferentes para este juego.</p>
                </div>
              </TransitionGroup>
            </section>

            <!-- SECTION 02: GAME DETAILS (NO SEDE) -->
            <section class="form-section">
              <div class="section-heading"><span>02</span><div><h2>Datos de la jornada</h2><p>Define la fecha y subtítulo</p></div></div>
              <div class="field-grid">
                <label class="input-field"><span>Fecha</span><div class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M6 2v3M14 2v3M3.5 8h13M5 4h10a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2z" fill="none" stroke="currentColor" stroke-width="1.6" /></svg><input v-model="resultsDate" type="date" required /></div></label>
                <label class="input-field"><span>Subtítulo <small>Opcional</small></span><div class="input-shell"><svg viewBox="0 0 20 20" aria-hidden="true"><path d="M4 6h12M4 10h8M4 14h10" fill="none" stroke="currentColor" stroke-width="1.6" /></svg><input v-model="resultsEyebrow" type="text" placeholder="TEMPORADA REGULAR" /></div></label>
              </div>
            </section>

            <!-- SECTION 03: BACKGROUNDS & THEMES -->
            <section class="form-section compact-section">
              <div class="section-heading theme-heading"><span>03</span><div><h2>Fondo y Estilo</h2><p>Personaliza el diseño visual</p></div></div>
              <div class="bg-options">
                <label v-for="bg in resultsBackgrounds" :key="bg.id" :class="{ active: resultsBgId === bg.id }">
                  <input v-model="resultsBgId" type="radio" :value="bg.id" name="res-background" />
                  <span class="bg-thumb" :style="{ backgroundImage: `url(${bg.asset})` }"></span>
                  <span>{{ bg.name }}</span>
                  <svg viewBox="0 0 16 16" aria-hidden="true"><path d="m3 8 3.5 3.5L13 5" fill="none" stroke="currentColor" stroke-width="2" /></svg>
                </label>
              </div>
              <div class="theme-options" style="margin-top: 10px;">
                <label v-for="theme in resultsThemes" :key="theme.id" :class="{ active: resultsThemeId === theme.id }">
                  <input v-model="resultsThemeId" type="radio" :value="theme.id" name="res-theme" />
                  <i :style="{ background: theme.accent }"></i>
                  <span>{{ theme.name }}</span>
                  <svg viewBox="0 0 16 16" aria-hidden="true"><path d="m3 8 3.5 3.5L13 5" fill="none" stroke="currentColor" stroke-width="2" /></svg>
                </label>
              </div>
            </section>

            <!-- ACTIONS -->
            <div class="form-actions">
              <button class="download-button" type="submit" :disabled="!isResultsFormComplete || isExporting">
                <span v-if="isExporting" class="button-spinner" aria-hidden="true"></span>
                <svg v-else viewBox="0 0 20 20" aria-hidden="true"><path d="M10 3v9m0 0 3.5-3.5M10 12 6.5 8.5M3 14.5v1a1.5 1.5 0 0 0 1.5 1.5h11a1.5 1.5 0 0 0 1.5-1.5v-1" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round" /></svg>
                <span>{{ isExporting ? 'Generando pizarra...' : 'Descargar Pizarra HD (PNG)' }}</span>
              </button>
              <p><svg viewBox="0 0 16 16" aria-hidden="true"><path d="M8 2a6 6 0 1 0 0 12A6 6 0 0 0 8 2Zm0 3v3.5l2.5 1.5" fill="none" stroke="currentColor" stroke-width="1.4" /></svg>Pizarra de resultados optimizada (1080 × 1080 px)</p>
            </div>
          </form>
        </template>
      </aside>

      <!-- ============================================== -->
      <!-- PREVIEW PANEL                                  -->
      <!-- ============================================== -->
      <section class="preview-panel" aria-label="Previsualización del afiche">
        <div class="preview-toolbar">
          <div>
            <h2>{{ currentView === 'schedule' ? 'Previsualización de Cartelera' : 'Previsualización de Pizarra de Resultados' }}</h2>
            <p>{{ currentView === 'schedule' ? 'Diseño en tiempo real para redes sociales' : 'Marcadores oficiales R · H · E' }}</p>
          </div>
          <span class="format-pill">1080 × 1080</span>
        </div>

        <div class="poster-stage">
          <div class="poster-shadow"></div>

          <!-- ============================================================== -->
          <!-- POSTER SVG: SCHEDULE VIEW                                      -->
          <!-- ============================================================== -->
          <svg
            v-if="currentView === 'schedule'"
            ref="posterSvg"
            class="poster"
            viewBox="0 0 1080 1080"
            role="img"
            aria-label="Cartel de juego de softball"
          >
            <defs>
              <pattern id="grid" width="40" height="40" patternUnits="userSpaceOnUse"><path d="M40 0H0v40" fill="none" stroke="#FFFFFF" stroke-opacity=".03" /></pattern>
              <linearGradient id="cardFill" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="#07131D" stop-opacity=".96" /><stop offset="100%" stop-color="#040A0F" stop-opacity=".98" /></linearGradient>
              <linearGradient id="accentLine" x1="0" y1="0" x2="1" y2="0"><stop offset="0%" :stop-color="selectedTheme.accent" /><stop offset="50%" stop-color="#FFFFFF" /><stop offset="100%" :stop-color="selectedTheme.accent" /></linearGradient>
              <filter id="softShadow" x="-10%" y="-10%" width="120%" height="130%"><feDropShadow dx="0" dy="16" stdDeviation="20" flood-color="#000000" flood-opacity=".45" /></filter>
              <filter id="logoShadow" x="-20%" y="-20%" width="140%" height="140%"><feDropShadow dx="0" dy="8" stdDeviation="12" flood-color="#000000" flood-opacity=".55" /></filter>
              <clipPath id="leagueLogoClip"><rect x="0" y="0" width="44" height="44" rx="10" /></clipPath>
              <clipPath id="logo-away"><circle cx="0" cy="0" r="76" /></clipPath>
              <clipPath id="logo-home"><circle cx="0" cy="0" r="76" /></clipPath>
              <clipPath id="crestClip2"><circle cx="0" cy="0" r="40" /></clipPath>
              <clipPath id="crestClip3"><circle cx="0" cy="0" r="30" /></clipPath>
              <clipPath id="crestClip4"><circle cx="0" cy="0" r="24" /></clipPath>
            </defs>

            <g>
              <image :href="bgDataUrls[selectedBgId] || currentBg.asset" x="0" y="0" width="1080" height="1080" preserveAspectRatio="xMidYMid slice" />
              <rect width="1080" height="1080" fill="#03070A" fill-opacity=".75" />
              <rect width="1080" height="1080" fill="url(#grid)" />
              <circle cx="540" cy="540" r="460" fill="none" stroke="#FFFFFF" stroke-opacity=".06" stroke-width="1" />

              <!-- HEADER -->
              <g transform="translate(72 58)">
                <rect x="0" y="0" width="44" height="44" rx="11" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />
                <image :href="leagueLogoDataUrl || logoLigaAsset" x="0" y="0" width="44" height="44" preserveAspectRatio="xMidYMid slice" clip-path="url(#leagueLogoClip)" />
                <text x="60" y="19" fill="#FFFFFF" font-size="23" font-weight="800" letter-spacing="3">LIGA CRISTIANA</text>
                <text x="60" y="39" fill="#FFFFFF" fill-opacity=".52" font-size="11" font-weight="600" letter-spacing="3.2">JUGANDO PARA CRISTO</text>
              </g>

              <g transform="translate(1008 61)" text-anchor="end"><text y="15" fill="#FFFFFF" fill-opacity=".54" font-size="11" font-weight="700" letter-spacing="2.5">JORNADA DE JUEGO</text><text y="38" fill="#FFFFFF" font-size="18" font-weight="800" letter-spacing="2">{{ gameYear }}</text></g>
              <path d="M72 122H1008" stroke="#FFFFFF" stroke-opacity=".17" /><path d="M72 122H245" :stroke="selectedTheme.accent" stroke-width="3" />
              <text x="540" y="174" text-anchor="middle" :fill="selectedTheme.accentSoft" font-size="15" font-weight="800" letter-spacing="6.2">{{ eyebrowDisplay }}</text>
              <text x="540" y="229" text-anchor="middle" fill="#FFFFFF" font-size="53" font-weight="900" letter-spacing="5">{{ computedPosterTitle }}</text>
              <text x="540" y="267" text-anchor="middle" fill="#FFFFFF" fill-opacity=".68" font-size="17" font-weight="700" letter-spacing="2.7">{{ formattedDate }}</text>

              <!-- LAYOUT 1: SINGLE MATCH -->
              <template v-if="matches.length === 1">
                <g filter="url(#softShadow)"><rect x="72" y="302" width="936" height="496" rx="26" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" /><path d="M72 363H1008" stroke="#FFFFFF" stroke-opacity=".12" /><path d="M540 363V770" stroke="#FFFFFF" stroke-opacity=".07" /><path d="M72 302H1008" :stroke="selectedTheme.accent" stroke-width="3" opacity=".9" /></g>

                <g v-for="item in posterTeamsSingle" :key="item.slot" :transform="`translate(${item.x} 0)`">
                  <text x="0" y="341" text-anchor="middle" fill="#FFFFFF" fill-opacity=".48" font-size="13" font-weight="800" letter-spacing="4.5">{{ item.label }}</text>
                  <g transform="translate(0 488)" filter="url(#logoShadow)">
                    <circle r="105" fill="#02080C" fill-opacity=".85" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                    <circle r="95" :fill="item.team.primary" />
                    <circle r="88" fill="#FFFFFF" fill-opacity=".08" />
                    <image :href="teamDataUrls[item.team.id] || item.team.logo" x="-76" y="-76" width="152" height="152" preserveAspectRatio="xMidYMid meet" :clip-path="`url(#logo-${item.slot})`" />
                  </g>
                  <text x="0" text-anchor="middle" fill="#FFFFFF" :font-size="formatTeamName(item.team.name).fontSize" font-weight="900" letter-spacing="1">
                    <tspan v-for="(line, lIdx) in formatTeamName(item.team.name).lines" :key="lIdx" x="0" :y="formatTeamName(item.team.name).startY + lIdx * formatTeamName(item.team.name).lineHeight">{{ line }}</tspan>
                  </text>
                  <text x="0" :y="formatTeamName(item.team.name).cityY" text-anchor="middle" :fill="selectedTheme.accentSoft" font-size="14" font-weight="700" letter-spacing="4.5">{{ item.team.city.toLocaleUpperCase('es-VE') }}</text>
                </g>

                <g transform="translate(540 500)"><circle r="53" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" /><circle r="43" :fill="selectedTheme.accent" /><path d="M-29-24Q0-4 29-24M-29 24Q0 4 29 24" fill="none" stroke="#07131D" stroke-opacity=".45" stroke-width="2.5" /><text y="7" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="27" font-weight="900" letter-spacing="-1">VS</text></g>
              </template>

              <!-- LAYOUT 2: TWO MATCHES -->
              <template v-else-if="matches.length === 2">
                <g v-for="(m, mIdx) in matches" :key="m.id" :transform="`translate(72 ${300 + mIdx * 256})`">
                  <g filter="url(#softShadow)"><rect width="936" height="240" rx="22" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" /><rect width="6" height="240" rx="3" :fill="selectedTheme.accent" /></g>

                  <!-- AWAY TEAM -->
                  <g transform="translate(195 120)">
                    <g transform="translate(-105 0)" filter="url(#logoShadow)">
                      <circle r="56" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                      <circle r="50" :fill="getTeam(m.awayTeamId).primary" />
                      <image :href="teamDataUrls[m.awayTeamId] || getTeam(m.awayTeamId).logo" x="-40" y="-40" width="80" height="80" preserveAspectRatio="xMidYMid meet" clip-path="url(#crestClip2)" />
                    </g>
                    <text x="-32" fill="#FFFFFF" :font-size="formatMultiName(getTeam(m.awayTeamId).name, 'two').fontSize" font-weight="900" letter-spacing=".5">
                      <tspan v-for="(line, lIdx) in formatMultiName(getTeam(m.awayTeamId).name, 'two').lines" :key="lIdx" x="-32" :y="formatMultiName(getTeam(m.awayTeamId).name, 'two').startY + lIdx * formatMultiName(getTeam(m.awayTeamId).name, 'two').lineHeight">{{ line }}</tspan>
                    </text>
                    <text x="-32" :y="formatMultiName(getTeam(m.awayTeamId).name, 'two').labelY" :fill="selectedTheme.accentSoft" font-size="12" font-weight="700" letter-spacing="2">VISITANTE</text>
                  </g>

                  <!-- VS CENTER -->
                  <g transform="translate(468 96)">
                    <circle r="38" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                    <circle r="30" :fill="selectedTheme.accent" />
                    <text y="5" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="18" font-weight="900">VS</text>
                    <text y="68" text-anchor="middle" fill="#FFFFFF" fill-opacity=".92" font-size="18" font-weight="900" letter-spacing=".5">{{ formatTime(m.time) }}</text>
                  </g>

                  <!-- HOME TEAM -->
                  <g transform="translate(635 120)">
                    <text x="140" text-anchor="end" fill="#FFFFFF" :font-size="formatMultiName(getTeam(m.homeTeamId).name, 'two').fontSize" font-weight="900" letter-spacing=".5">
                      <tspan v-for="(line, lIdx) in formatMultiName(getTeam(m.homeTeamId).name, 'two').lines" :key="lIdx" x="140" :y="formatMultiName(getTeam(m.homeTeamId).name, 'two').startY + lIdx * formatMultiName(getTeam(m.homeTeamId).name, 'two').lineHeight">{{ line }}</tspan>
                    </text>
                    <text x="140" :y="formatMultiName(getTeam(m.homeTeamId).name, 'two').labelY" text-anchor="end" :fill="selectedTheme.accentSoft" font-size="12" font-weight="700" letter-spacing="2">LOCAL</text>
                    <g transform="translate(210 0)" filter="url(#logoShadow)">
                      <circle r="56" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                      <circle r="50" :fill="getTeam(m.homeTeamId).primary" />
                      <image :href="teamDataUrls[m.homeTeamId] || getTeam(m.homeTeamId).logo" x="-40" y="-40" width="80" height="80" preserveAspectRatio="xMidYMid meet" clip-path="url(#crestClip2)" />
                    </g>
                  </g>
                </g>
              </template>

              <!-- LAYOUT 3: THREE MATCHES -->
              <template v-else-if="matches.length === 3">
                <g v-for="(m, mIdx) in matches" :key="m.id" :transform="`translate(72 ${300 + mIdx * 170})`">
                  <g filter="url(#softShadow)"><rect width="936" height="154" rx="18" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" /><rect width="5" height="154" rx="2.5" :fill="selectedTheme.accent" /></g>

                  <!-- AWAY -->
                  <g transform="translate(68 78)">
                    <g filter="url(#logoShadow)">
                      <circle r="44" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="39" :fill="getTeam(m.awayTeamId).primary" />
                      <image :href="teamDataUrls[m.awayTeamId] || getTeam(m.awayTeamId).logo" x="-30" y="-30" width="60" height="60" preserveAspectRatio="xMidYMid meet" clip-path="url(#crestClip3)" />
                    </g>
                    <text x="58" fill="#FFFFFF" :font-size="formatMultiName(getTeam(m.awayTeamId).name, 'three').fontSize" font-weight="900" letter-spacing=".4">
                      <tspan v-for="(line, lIdx) in formatMultiName(getTeam(m.awayTeamId).name, 'three').lines" :key="lIdx" x="58" :y="formatMultiName(getTeam(m.awayTeamId).name, 'three').startY + lIdx * formatMultiName(getTeam(m.awayTeamId).name, 'three').lineHeight">{{ line }}</tspan>
                    </text>
                  </g>

                  <!-- VS CENTER -->
                  <g transform="translate(468 62)">
                    <circle r="28" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="2" />
                    <circle r="22" :fill="selectedTheme.accent" />
                    <text y="4" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="14" font-weight="900">VS</text>
                    <text y="52" text-anchor="middle" fill="#FFFFFF" fill-opacity=".92" font-size="16" font-weight="900" letter-spacing=".5">{{ formatTime(m.time) }}</text>
                  </g>

                  <!-- HOME -->
                  <g transform="translate(868 78)">
                    <text x="-58" text-anchor="end" fill="#FFFFFF" :font-size="formatMultiName(getTeam(m.homeTeamId).name, 'three').fontSize" font-weight="900" letter-spacing=".4">
                      <tspan v-for="(line, lIdx) in formatMultiName(getTeam(m.homeTeamId).name, 'three').lines" :key="lIdx" x="-58" :y="formatMultiName(getTeam(m.homeTeamId).name, 'three').startY + lIdx * formatMultiName(getTeam(m.homeTeamId).name, 'three').lineHeight">{{ line }}</tspan>
                    </text>
                    <g filter="url(#logoShadow)">
                      <circle r="44" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="39" :fill="getTeam(m.homeTeamId).primary" />
                      <image :href="teamDataUrls[m.homeTeamId] || getTeam(m.homeTeamId).logo" x="-30" y="-30" width="60" height="60" preserveAspectRatio="xMidYMid meet" clip-path="url(#crestClip3)" />
                    </g>
                  </g>
                </g>
              </template>

              <!-- LAYOUT 4: FOUR MATCHES -->
              <template v-else>
                <g v-for="(m, mIdx) in matches" :key="m.id" :transform="`translate(72 ${300 + mIdx * 128})`">
                  <g filter="url(#softShadow)"><rect width="936" height="114" rx="16" fill="url(#cardFill)" stroke="#FFFFFF" stroke-opacity=".15" /><rect width="5" height="114" rx="2.5" :fill="selectedTheme.accent" /></g>

                  <!-- AWAY -->
                  <g transform="translate(68 57)">
                    <g filter="url(#logoShadow)">
                      <circle r="34" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="30" :fill="getTeam(m.awayTeamId).primary" />
                      <image :href="teamDataUrls[m.awayTeamId] || getTeam(m.awayTeamId).logo" x="-24" y="-24" width="48" height="48" preserveAspectRatio="xMidYMid meet" clip-path="url(#crestClip4)" />
                    </g>
                    <text x="48" fill="#FFFFFF" :font-size="formatMultiName(getTeam(m.awayTeamId).name, 'four').fontSize" font-weight="900" letter-spacing=".3">
                      <tspan v-for="(line, lIdx) in formatMultiName(getTeam(m.awayTeamId).name, 'four').lines" :key="lIdx" x="48" :y="formatMultiName(getTeam(m.awayTeamId).name, 'four').startY + lIdx * formatMultiName(getTeam(m.awayTeamId).name, 'four').lineHeight">{{ line }}</tspan>
                    </text>
                  </g>

                  <!-- VS CENTER -->
                  <g transform="translate(468 44)">
                    <circle r="22" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.8" />
                    <circle r="17" :fill="selectedTheme.accent" />
                    <text y="4" text-anchor="middle" dominant-baseline="middle" fill="#07131D" font-size="11" font-weight="900">VS</text>
                    <text y="42" text-anchor="middle" fill="#FFFFFF" fill-opacity=".92" font-size="14" font-weight="900" letter-spacing=".5">{{ formatTime(m.time) }}</text>
                  </g>

                  <!-- HOME -->
                  <g transform="translate(868 57)">
                    <text x="-48" text-anchor="end" fill="#FFFFFF" :font-size="formatMultiName(getTeam(m.homeTeamId).name, 'four').fontSize" font-weight="900" letter-spacing=".3">
                      <tspan v-for="(line, lIdx) in formatMultiName(getTeam(m.homeTeamId).name, 'four').lines" :key="lIdx" x="-48" :y="formatMultiName(getTeam(m.homeTeamId).name, 'four').startY + lIdx * formatMultiName(getTeam(m.homeTeamId).name, 'four').lineHeight">{{ line }}</tspan>
                    </text>
                    <g filter="url(#logoShadow)">
                      <circle r="34" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.5" />
                      <circle r="30" :fill="getTeam(m.homeTeamId).primary" />
                      <image :href="teamDataUrls[m.homeTeamId] || getTeam(m.homeTeamId).logo" x="-24" y="-24" width="48" height="48" preserveAspectRatio="xMidYMid meet" clip-path="url(#crestClip4)" />
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

          <!-- ============================================================== -->
          <!-- POSTER SVG: RESULTS SCOREBOARD TABLES WITH R · H · E GRID      -->
          <!-- ============================================================== -->
          <svg
            v-else
            ref="posterSvg"
            class="poster"
            viewBox="0 0 1080 1080"
            role="img"
            aria-label="Pizarra de resultados de softball"
          >
            <defs>
              <pattern id="resGrid" width="40" height="40" patternUnits="userSpaceOnUse"><path d="M40 0H0v40" fill="none" stroke="#FFFFFF" stroke-opacity=".03" /></pattern>
              <linearGradient id="resCardFill" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="#08141F" stop-opacity=".97" /><stop offset="100%" stop-color="#04090E" stop-opacity=".99" /></linearGradient>
              <linearGradient id="resRowWinner" x1="0" y1="0" x2="1" y2="0"><stop offset="0%" :stop-color="selectedResultsTheme.accent" stop-opacity=".22" /><stop offset="100%" :stop-color="selectedResultsTheme.accent" stop-opacity=".04" /></linearGradient>
              <linearGradient id="resAccentLine" x1="0" y1="0" x2="1" y2="0"><stop offset="0%" :stop-color="selectedResultsTheme.accent" /><stop offset="50%" stop-color="#FFFFFF" /><stop offset="100%" :stop-color="selectedResultsTheme.accent" /></linearGradient>
              <filter id="resSoftShadow" x="-10%" y="-10%" width="120%" height="130%"><feDropShadow dx="0" dy="16" stdDeviation="20" flood-color="#000000" flood-opacity=".45" /></filter>
              <filter id="resLogoShadow" x="-20%" y="-20%" width="140%" height="140%"><feDropShadow dx="0" dy="8" stdDeviation="12" flood-color="#000000" flood-opacity=".55" /></filter>
              <clipPath id="resLeagueLogoClip"><rect x="0" y="0" width="44" height="44" rx="11" /></clipPath>
              <clipPath id="resCrestClip1"><circle cx="0" cy="0" r="46" /></clipPath>
              <clipPath id="resCrestClip2"><circle cx="0" cy="0" r="32" /></clipPath>
              <clipPath id="resCrestClip3"><circle cx="0" cy="0" r="25" /></clipPath>
              <clipPath id="resCrestClip4"><circle cx="0" cy="0" r="19" /></clipPath>
            </defs>

            <g>
              <image :href="bgDataUrls[resultsBgId] || currentResultsBg.asset" x="0" y="0" width="1080" height="1080" preserveAspectRatio="xMidYMid slice" />
              <rect width="1080" height="1080" fill="#03070A" fill-opacity=".78" />
              <rect width="1080" height="1080" fill="url(#resGrid)" />
              <circle cx="540" cy="540" r="460" fill="none" stroke="#FFFFFF" stroke-opacity=".06" stroke-width="1" />

              <!-- HEADER -->
              <g transform="translate(72 58)">
                <rect x="0" y="0" width="44" height="44" rx="11" fill="#07131D" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />
                <image :href="leagueLogoDataUrl || logoLigaAsset" x="0" y="0" width="44" height="44" preserveAspectRatio="xMidYMid slice" clip-path="url(#resLeagueLogoClip)" />
                <text x="60" y="19" fill="#FFFFFF" font-size="23" font-weight="800" letter-spacing="3">LIGA CRISTIANA</text>
                <text x="60" y="39" fill="#FFFFFF" fill-opacity=".52" font-size="11" font-weight="600" letter-spacing="3.2">JUGANDO PARA CRISTO</text>
              </g>

              <g transform="translate(1008 61)" text-anchor="end"><text y="15" fill="#FFFFFF" fill-opacity=".54" font-size="11" font-weight="700" letter-spacing="2.5">PIZARRA OFICIAL</text><text y="38" fill="#FFFFFF" font-size="18" font-weight="800" letter-spacing="2">{{ resultsGameYear }}</text></g>
              <path d="M72 122H1008" stroke="#FFFFFF" stroke-opacity=".17" /><path d="M72 122H245" :stroke="selectedResultsTheme.accent" stroke-width="3" />
              <text x="540" y="174" text-anchor="middle" :fill="selectedResultsTheme.accentSoft" font-size="15" font-weight="800" letter-spacing="6.2">{{ resultsEyebrowDisplay }}</text>
              <text x="540" y="229" text-anchor="middle" fill="#FFFFFF" font-size="52" font-weight="900" letter-spacing="4.5">{{ computedResultsTitle }}</text>
              <text x="540" y="267" text-anchor="middle" fill="#FFFFFF" fill-opacity=".68" font-size="17" font-weight="700" letter-spacing="2.7">{{ formattedResultsDate }}</text>

              <!-- ============================================== -->
              <!-- RESULTS LAYOUT 1: SINGLE MATCH GRID TABLE      -->
              <!-- ============================================== -->
              <template v-if="resultMatches.length === 1">
                <g transform="translate(84 300)" filter="url(#resSoftShadow)">
                  <!-- OUTER TABLE CONTAINER -->
                  <rect width="912" :height="resultMatches[0].note ? 568 : 520" rx="18" fill="url(#resCardFill)" stroke="#FFFFFF" stroke-opacity=".22" stroke-width="1.8" />
                  
                  <!-- COLUMN R (RUNS) ACCENT HIGHLIGHT STRIP -->
                  <rect x="530" y="0" width="130" :height="resultMatches[0].note ? 568 : 520" :fill="selectedResultsTheme.accent" fill-opacity=".06" />

                  <!-- TABLE HEADER ROW (y: 0 to 56) -->
                  <rect width="912" height="56" rx="18" fill="#040A0F" fill-opacity=".95" />
                  <rect y="36" width="912" height="20" fill="#040A0F" fill-opacity=".95" />
                  <line x1="0" y1="56" x2="912" y2="56" stroke="#FFFFFF" stroke-opacity=".25" stroke-width="1.6" />

                  <!-- HEADER LABELS (ENGLISH STATS) -->
                  <text x="32" y="35" fill="#FFFFFF" font-size="16" font-weight="900" letter-spacing="2.5">JUEGO 1 · <tspan :fill="selectedResultsTheme.accent">{{ resultMatches[0].status || 'FINAL' }}</tspan></text>
                  <text x="595" y="36" text-anchor="middle" :fill="selectedResultsTheme.accent" font-size="16.5" font-weight="900" letter-spacing="2">R (RUNS)</text>
                  <text x="720" y="36" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="16.5" font-weight="900" letter-spacing="2">H (HITS)</text>
                  <text x="846" y="36" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="16.5" font-weight="900" letter-spacing="2">E (ERRORS)</text>

                  <!-- VERTICAL GRID LINES -->
                  <line x1="530" y1="0" x2="530" :y2="resultMatches[0].note ? 568 : 520" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />
                  <line x1="660" y1="0" x2="660" :y2="resultMatches[0].note ? 568 : 520" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />
                  <line x1="780" y1="0" x2="780" :y2="resultMatches[0].note ? 568 : 520" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />

                  <!-- HORIZONTAL ROW DIVIDER (y=288) -->
                  <line x1="0" y1="288" x2="912" y2="288" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />

                  <!-- ROW 1: TEAM 1 (y: 56 to 288) -->
                  <rect v-if="resultMatches[0].awayScore.runs > resultMatches[0].homeScore.runs" x="0" y="56" width="912" height="232" fill="url(#resRowWinner)" />
                  <g transform="translate(68 172)" filter="url(#resLogoShadow)">
                    <circle r="46" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="2" />
                    <circle r="40" :fill="getTeam(resultMatches[0].awayTeamId).primary" />
                    <image :href="teamDataUrls[resultMatches[0].awayTeamId] || getTeam(resultMatches[0].awayTeamId).logo" x="-34" y="-34" width="68" height="68" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip1)" />
                  </g>
                  <text x="136" :y="resultMatches[0].awayScore.runs > resultMatches[0].homeScore.runs ? 162 : 178" fill="#FFFFFF" font-size="29" font-weight="900" letter-spacing=".5">{{ getTeam(resultMatches[0].awayTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text v-if="resultMatches[0].awayScore.runs > resultMatches[0].homeScore.runs" x="136" y="195" fill="#FDE047" font-size="13" font-weight="800" letter-spacing="3">★ GANADOR</text>
                  
                  <text x="595" y="188" text-anchor="middle" font-size="58" font-weight="900" :fill="resultMatches[0].awayScore.runs > resultMatches[0].homeScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'" letter-spacing="-1">{{ resultMatches[0].awayScore.runs }}</text>
                  <text x="720" y="188" text-anchor="middle" font-size="58" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ resultMatches[0].awayScore.hits }}</text>
                  <text x="846" y="188" text-anchor="middle" font-size="58" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ resultMatches[0].awayScore.errors }}</text>

                  <!-- ROW 2: TEAM 2 (y: 288 to 520) -->
                  <rect v-if="resultMatches[0].homeScore.runs > resultMatches[0].awayScore.runs" x="0" y="288" width="912" height="232" :rx="resultMatches[0].note ? 0 : 18" fill="url(#resRowWinner)" />
                  <g transform="translate(68 404)" filter="url(#resLogoShadow)">
                    <circle r="46" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="2" />
                    <circle r="40" :fill="getTeam(resultMatches[0].homeTeamId).primary" />
                    <image :href="teamDataUrls[resultMatches[0].homeTeamId] || getTeam(resultMatches[0].homeTeamId).logo" x="-34" y="-34" width="68" height="68" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip1)" />
                  </g>
                  <text x="136" :y="resultMatches[0].homeScore.runs > resultMatches[0].awayScore.runs ? 394 : 410" fill="#FFFFFF" font-size="29" font-weight="900" letter-spacing=".5">{{ getTeam(resultMatches[0].homeTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text v-if="resultMatches[0].homeScore.runs > resultMatches[0].awayScore.runs" x="136" y="427" fill="#FDE047" font-size="13" font-weight="800" letter-spacing="3">★ GANADOR</text>

                  <text x="595" y="420" text-anchor="middle" font-size="58" font-weight="900" :fill="resultMatches[0].homeScore.runs > resultMatches[0].awayScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'" letter-spacing="-1">{{ resultMatches[0].homeScore.runs }}</text>
                  <text x="720" y="420" text-anchor="middle" font-size="58" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ resultMatches[0].homeScore.hits }}</text>
                  <text x="846" y="420" text-anchor="middle" font-size="58" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ resultMatches[0].homeScore.errors }}</text>

                  <!-- INTEGRATED NOTE FOOTER ROW -->
                  <template v-if="resultMatches[0].note">
                    <line x1="0" y1="520" x2="912" y2="520" stroke="#FFFFFF" stroke-opacity=".22" stroke-width="1.5" />
                    <rect y="520" width="912" height="48" rx="18" fill="#04090E" fill-opacity=".96" />
                    <rect y="520" width="912" height="20" fill="#04090E" fill-opacity=".96" />
                    <rect x="0" y="528" width="5" height="32" rx="2.5" fill="#FDE047" />
                    <text x="22" y="550" fill="#FDE047" font-size="13" font-weight="900" letter-spacing="2">
                      NOTA: <tspan fill="#FFFFFF" fill-opacity=".95" font-weight="700">{{ resultMatches[0].note.toLocaleUpperCase('es-VE') }}</tspan>
                    </text>
                  </template>
                </g>
              </template>

              <!-- ============================================== -->
              <!-- RESULTS LAYOUT 2: TWO MATCHES GRID TABLES      -->
              <!-- ============================================== -->
              <template v-else-if="resultMatches.length === 2">
                <g v-for="(rm, rmIdx) in resultMatches" :key="rm.id" :transform="`translate(84 ${300 + rmIdx * 330})`" filter="url(#resSoftShadow)">
                  <!-- OUTER CONTAINER -->
                  <rect width="912" :height="rm.note ? 302 : 268" rx="16" fill="url(#resCardFill)" stroke="#FFFFFF" stroke-opacity=".22" stroke-width="1.6" />
                  <rect x="530" y="0" width="130" :height="rm.note ? 302 : 268" :fill="selectedResultsTheme.accent" fill-opacity=".06" />

                  <!-- HEADER ROW -->
                  <rect width="912" height="46" rx="16" fill="#040A0F" fill-opacity=".95" />
                  <rect y="26" width="912" height="20" fill="#040A0F" fill-opacity=".95" />
                  <line x1="0" y1="46" x2="912" y2="46" stroke="#FFFFFF" stroke-opacity=".22" stroke-width="1.4" />

                  <text x="24" y="30" fill="#FFFFFF" font-size="15" font-weight="900" letter-spacing="2.5">JUEGO {{ rmIdx + 1 }} · <tspan :fill="selectedResultsTheme.accent">{{ rm.status || 'FINAL' }}</tspan></text>
                  <text x="595" y="30" text-anchor="middle" :fill="selectedResultsTheme.accent" font-size="16" font-weight="900" letter-spacing="2">R</text>
                  <text x="720" y="30" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="16" font-weight="900" letter-spacing="2">H</text>
                  <text x="846" y="30" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="16" font-weight="900" letter-spacing="2">E</text>

                  <!-- VERTICAL LINES -->
                  <line x1="530" y1="0" x2="530" :y2="rm.note ? 302 : 268" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.4" />
                  <line x1="660" y1="0" x2="660" :y2="rm.note ? 302 : 268" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.4" />
                  <line x1="780" y1="0" x2="780" :y2="rm.note ? 302 : 268" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.4" />

                  <!-- ROW DIVIDER (y=157) -->
                  <line x1="0" y1="157" x2="912" y2="157" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />

                  <!-- ROW 1: TEAM 1 -->
                  <rect v-if="rm.awayScore.runs > rm.homeScore.runs" x="0" y="46" width="912" height="111" fill="url(#resRowWinner)" />
                  <g transform="translate(52 101)" filter="url(#resLogoShadow)">
                    <circle r="34" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />
                    <circle r="30" :fill="getTeam(rm.awayTeamId).primary" />
                    <image :href="teamDataUrls[rm.awayTeamId] || getTeam(rm.awayTeamId).logo" x="-25" y="-25" width="50" height="50" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip2)" />
                  </g>
                  <text x="100" :y="rm.awayScore.runs > rm.homeScore.runs ? 98 : 108" fill="#FFFFFF" font-size="24" font-weight="900" letter-spacing=".5">{{ getTeam(rm.awayTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text v-if="rm.awayScore.runs > rm.homeScore.runs" x="100" y="121" fill="#FDE047" font-size="11.5" font-weight="800" letter-spacing="2">★ GANADOR</text>
                  <text x="595" y="110" text-anchor="middle" font-size="40" font-weight="900" :fill="rm.awayScore.runs > rm.homeScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'" letter-spacing="-1">{{ rm.awayScore.runs }}</text>
                  <text x="720" y="110" text-anchor="middle" font-size="40" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ rm.awayScore.hits }}</text>
                  <text x="846" y="110" text-anchor="middle" font-size="40" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ rm.awayScore.errors }}</text>

                  <!-- ROW 2: TEAM 2 -->
                  <rect v-if="rm.homeScore.runs > rm.awayScore.runs" x="0" y="157" width="912" height="111" :rx="rm.note ? 0 : 16" fill="url(#resRowWinner)" />
                  <g transform="translate(52 212)" filter="url(#resLogoShadow)">
                    <circle r="34" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.5" />
                    <circle r="30" :fill="getTeam(rm.homeTeamId).primary" />
                    <image :href="teamDataUrls[rm.homeTeamId] || getTeam(rm.homeTeamId).logo" x="-25" y="-25" width="50" height="50" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip2)" />
                  </g>
                  <text x="100" :y="rm.homeScore.runs > rm.awayScore.runs ? 209 : 219" fill="#FFFFFF" font-size="24" font-weight="900" letter-spacing=".5">{{ getTeam(rm.homeTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text v-if="rm.homeScore.runs > rm.awayScore.runs" x="100" y="232" fill="#FDE047" font-size="11.5" font-weight="800" letter-spacing="2">★ GANADOR</text>
                  <text x="595" y="221" text-anchor="middle" font-size="40" font-weight="900" :fill="rm.homeScore.runs > rm.awayScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'" letter-spacing="-1">{{ rm.homeScore.runs }}</text>
                  <text x="720" y="221" text-anchor="middle" font-size="40" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ rm.homeScore.hits }}</text>
                  <text x="846" y="221" text-anchor="middle" font-size="40" font-weight="900" fill="#FFFFFF" fill-opacity=".92" letter-spacing="-1">{{ rm.homeScore.errors }}</text>

                  <!-- INTEGRATED NOTE FOOTER ROW -->
                  <template v-if="rm.note">
                    <line x1="0" y1="268" x2="912" y2="268" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.3" />
                    <rect y="268" width="912" height="34" rx="16" fill="#04090E" fill-opacity=".96" />
                    <rect y="268" width="912" height="15" fill="#04090E" fill-opacity=".96" />
                    <rect x="0" y="274" width="4" height="22" rx="2" fill="#FDE047" />
                    <text x="18" y="290" fill="#FDE047" font-size="11.5" font-weight="900" letter-spacing="1.5">
                      NOTA: <tspan fill="#FFFFFF" fill-opacity=".95" font-weight="700">{{ rm.note.toLocaleUpperCase('es-VE') }}</tspan>
                    </text>
                  </template>
                </g>
              </template>

              <!-- ============================================== -->
              <!-- RESULTS LAYOUT 3: THREE MATCHES GRID TABLES    -->
              <!-- ============================================== -->
              <template v-else-if="resultMatches.length === 3">
                <g v-for="(rm, rmIdx) in resultMatches" :key="rm.id" :transform="`translate(84 ${295 + rmIdx * 225})`" filter="url(#resSoftShadow)">
                  <!-- OUTER CONTAINER -->
                  <rect width="912" :height="rm.note ? 214 : 186" rx="14" fill="url(#resCardFill)" stroke="#FFFFFF" stroke-opacity=".22" stroke-width="1.5" />
                  <rect x="540" y="0" width="120" :height="rm.note ? 214 : 186" :fill="selectedResultsTheme.accent" fill-opacity=".06" />

                  <!-- HEADER -->
                  <rect width="912" height="36" rx="14" fill="#040A0F" fill-opacity=".95" />
                  <rect y="16" width="912" height="20" fill="#040A0F" fill-opacity=".95" />
                  <line x1="0" y1="36" x2="912" y2="36" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.3" />

                  <text x="20" y="24" fill="#FFFFFF" font-size="14.5" font-weight="900" letter-spacing="2">JUEGO {{ rmIdx + 1 }} · <tspan :fill="selectedResultsTheme.accent">{{ rm.status || 'FINAL' }}</tspan></text>
                  <text x="600" y="24" text-anchor="middle" :fill="selectedResultsTheme.accent" font-size="15" font-weight="900" letter-spacing="2">R</text>
                  <text x="720" y="24" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="15" font-weight="900" letter-spacing="2">H</text>
                  <text x="846" y="24" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="15" font-weight="900" letter-spacing="2">E</text>

                  <!-- VERTICAL LINES -->
                  <line x1="540" y1="0" x2="540" :y2="rm.note ? 214 : 186" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                  <line x1="660" y1="0" x2="660" :y2="rm.note ? 214 : 186" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                  <line x1="780" y1="0" x2="780" :y2="rm.note ? 214 : 186" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />

                  <!-- ROW DIVIDER (y=111) -->
                  <line x1="0" y1="111" x2="912" y2="111" stroke="#FFFFFF" stroke-opacity=".15" stroke-width="1.2" />

                  <!-- ROW 1: TEAM 1 -->
                  <rect v-if="rm.awayScore.runs > rm.homeScore.runs" x="0" y="36" width="912" height="75" fill="url(#resRowWinner)" />
                  <g transform="translate(42 73.5)" filter="url(#resLogoShadow)">
                    <circle r="26" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.3" />
                    <circle r="23" :fill="getTeam(rm.awayTeamId).primary" />
                    <image :href="teamDataUrls[rm.awayTeamId] || getTeam(rm.awayTeamId).logo" x="-19" y="-19" width="38" height="38" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip3)" />
                  </g>
                  <text x="80" y="80.5" fill="#FFFFFF" font-size="22" font-weight="900" letter-spacing=".4">{{ getTeam(rm.awayTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text x="600" y="82" text-anchor="middle" font-size="32" font-weight="900" :fill="rm.awayScore.runs > rm.homeScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'">{{ rm.awayScore.runs }}</text>
                  <text x="720" y="82" text-anchor="middle" font-size="32" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.awayScore.hits }}</text>
                  <text x="846" y="82" text-anchor="middle" font-size="32" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.awayScore.errors }}</text>

                  <!-- ROW 2: TEAM 2 -->
                  <rect v-if="rm.homeScore.runs > rm.awayScore.runs" x="0" y="111" width="912" height="75" :rx="rm.note ? 0 : 14" fill="url(#resRowWinner)" />
                  <g transform="translate(42 148.5)" filter="url(#resLogoShadow)">
                    <circle r="26" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.3" />
                    <circle r="23" :fill="getTeam(rm.homeTeamId).primary" />
                    <image :href="teamDataUrls[rm.homeTeamId] || getTeam(rm.homeTeamId).logo" x="-19" y="-19" width="38" height="38" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip3)" />
                  </g>
                  <text x="80" y="155.5" fill="#FFFFFF" font-size="22" font-weight="900" letter-spacing=".4">{{ getTeam(rm.homeTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text x="600" y="157" text-anchor="middle" font-size="32" font-weight="900" :fill="rm.homeScore.runs > rm.awayScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'">{{ rm.homeScore.runs }}</text>
                  <text x="720" y="157" text-anchor="middle" font-size="32" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.homeScore.hits }}</text>
                  <text x="846" y="157" text-anchor="middle" font-size="32" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.homeScore.errors }}</text>

                  <!-- INTEGRATED NOTE FOOTER ROW -->
                  <template v-if="rm.note">
                    <line x1="0" y1="186" x2="912" y2="186" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                    <rect y="186" width="912" height="28" rx="14" fill="#04090E" fill-opacity=".96" />
                    <rect y="186" width="912" height="12" fill="#04090E" fill-opacity=".96" />
                    <rect x="0" y="191" width="4" height="18" rx="2" fill="#FDE047" />
                    <text x="16" y="204.5" fill="#FDE047" font-size="10.5" font-weight="900" letter-spacing="1.2">
                      NOTA: <tspan fill="#FFFFFF" fill-opacity=".95" font-weight="700">{{ rm.note.toLocaleUpperCase('es-VE') }}</tspan>
                    </text>
                  </template>
                </g>
              </template>

              <!-- ============================================== -->
              <!-- RESULTS LAYOUT 4: FOUR MATCHES GRID TABLES     -->
              <!-- ============================================== -->
              <template v-else>
                <g v-for="(rm, rmIdx) in resultMatches" :key="rm.id" :transform="`translate(84 ${292 + rmIdx * 170})`" filter="url(#resSoftShadow)">
                  <!-- OUTER CONTAINER -->
                  <rect width="912" :height="rm.note ? 162 : 138" rx="12" fill="url(#resCardFill)" stroke="#FFFFFF" stroke-opacity=".22" stroke-width="1.4" />
                  <rect x="550" y="0" width="120" :height="rm.note ? 162 : 138" :fill="selectedResultsTheme.accent" fill-opacity=".06" />

                  <!-- HEADER -->
                  <rect width="912" height="30" rx="12" fill="#040A0F" fill-opacity=".95" />
                  <rect y="14" width="912" height="16" fill="#040A0F" fill-opacity=".95" />
                  <line x1="0" y1="30" x2="912" y2="30" stroke="#FFFFFF" stroke-opacity=".2" stroke-width="1.2" />

                  <text x="18" y="20" fill="#FFFFFF" font-size="13" font-weight="900" letter-spacing="2">JUEGO {{ rmIdx + 1 }} · <tspan :fill="selectedResultsTheme.accent">{{ rm.status || 'FINAL' }}</tspan></text>
                  <text x="610" y="20" text-anchor="middle" :fill="selectedResultsTheme.accent" font-size="13.5" font-weight="900" letter-spacing="1.5">R</text>
                  <text x="725" y="20" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="13.5" font-weight="900" letter-spacing="1.5">H</text>
                  <text x="846" y="20" text-anchor="middle" fill="#FFFFFF" fill-opacity=".75" font-size="13.5" font-weight="900" letter-spacing="1.5">E</text>

                  <!-- VERTICAL LINES -->
                  <line x1="550" y1="0" x2="550" :y2="rm.note ? 162 : 138" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                  <line x1="670" y1="0" x2="670" :y2="rm.note ? 162 : 138" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                  <line x1="780" y1="0" x2="780" :y2="rm.note ? 162 : 138" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />

                  <!-- ROW DIVIDER (y=84) -->
                  <line x1="0" y1="84" x2="912" y2="84" stroke="#FFFFFF" stroke-opacity=".15" stroke-width="1.1" />

                  <!-- ROW 1: TEAM 1 -->
                  <rect v-if="rm.awayScore.runs > rm.homeScore.runs" x="0" y="30" width="912" height="54" fill="url(#resRowWinner)" />
                  <g transform="translate(34 57)" filter="url(#resLogoShadow)">
                    <circle r="20" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                    <circle r="17" :fill="getTeam(rm.awayTeamId).primary" />
                    <image :href="teamDataUrls[rm.awayTeamId] || getTeam(rm.awayTeamId).logo" x="-14" y="-14" width="28" height="28" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip4)" />
                  </g>
                  <text x="62" y="63.5" fill="#FFFFFF" font-size="18.5" font-weight="900" letter-spacing=".4">{{ getTeam(rm.awayTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text x="610" y="64" text-anchor="middle" font-size="26" font-weight="900" :fill="rm.awayScore.runs > rm.homeScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'">{{ rm.awayScore.runs }}</text>
                  <text x="725" y="64" text-anchor="middle" font-size="26" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.awayScore.hits }}</text>
                  <text x="846" y="64" text-anchor="middle" font-size="26" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.awayScore.errors }}</text>

                  <!-- ROW 2: TEAM 2 -->
                  <rect v-if="rm.homeScore.runs > rm.awayScore.runs" x="0" y="84" width="912" height="54" :rx="rm.note ? 0 : 12" fill="url(#resRowWinner)" />
                  <g transform="translate(34 111)" filter="url(#resLogoShadow)">
                    <circle r="20" fill="#02080C" stroke="#FFFFFF" stroke-opacity=".18" stroke-width="1.2" />
                    <circle r="17" :fill="getTeam(rm.homeTeamId).primary" />
                    <image :href="teamDataUrls[rm.homeTeamId] || getTeam(rm.homeTeamId).logo" x="-14" y="-14" width="28" height="28" preserveAspectRatio="xMidYMid meet" clip-path="url(#resCrestClip4)" />
                  </g>
                  <text x="62" y="117.5" fill="#FFFFFF" font-size="18.5" font-weight="900" letter-spacing=".4">{{ getTeam(rm.homeTeamId).fullName.toLocaleUpperCase('es-VE') }}</text>
                  <text x="610" y="118" text-anchor="middle" font-size="26" font-weight="900" :fill="rm.homeScore.runs > rm.awayScore.runs ? selectedResultsTheme.accentSoft : '#FFFFFF'">{{ rm.homeScore.runs }}</text>
                  <text x="725" y="118" text-anchor="middle" font-size="26" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.homeScore.hits }}</text>
                  <text x="846" y="118" text-anchor="middle" font-size="26" font-weight="900" fill="#FFFFFF" fill-opacity=".92">{{ rm.homeScore.errors }}</text>

                  <!-- INTEGRATED NOTE FOOTER ROW -->
                  <template v-if="rm.note">
                    <line x1="0" y1="138" x2="912" y2="138" stroke="#FFFFFF" stroke-opacity=".16" stroke-width="1.1" />
                    <rect y="138" width="912" height="24" rx="12" fill="#04090E" fill-opacity=".96" />
                    <rect y="138" width="912" height="10" fill="#04090E" fill-opacity=".96" />
                    <rect x="0" y="142" width="3.5" height="16" rx="1.75" fill="#FDE047" />
                    <text x="14" y="154.5" fill="#FDE047" font-size="9.5" font-weight="900" letter-spacing=".9">
                      NOTA: <tspan fill="#FFFFFF" fill-opacity=".95" font-weight="700">{{ rm.note.toLocaleUpperCase('es-VE') }}</tspan>
                    </text>
                  </template>
                </g>
              </template>

              <!-- FOOTER MOTTO -->
              <g transform="translate(72 1030)"><text fill="#FFFFFF" fill-opacity=".42" font-size="11" font-weight="700" letter-spacing="2.4">SOFTBALL · PASIÓN · TRADICIÓN</text><text x="936" text-anchor="end" fill="#FFFFFF" fill-opacity=".42" font-size="11" font-weight="700" letter-spacing="2.4">#RESULTADOS</text></g>
              <rect x="0" y="1071" width="1080" height="9" :fill="selectedResultsTheme.accent" /><rect x="238" y="1071" width="604" height="9" fill="url(#resAccentLine)" />
            </g>
          </svg>
        </div>

        <div class="preview-footnote">
          <svg viewBox="0 0 20 20" aria-hidden="true"><path d="M3 5.5h14v9H3zM6 3v2.5M14 3v2.5M6 14.5V17h8v-2.5" fill="none" stroke="currentColor" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round" /></svg>
          <span><strong>Alta resolución.</strong> El archivo se exportará en formato PNG de 1080 × 1080 px listo para publicar.</span>
        </div>
      </section>
    </main>

    <Transition name="toast">
      <div v-if="notice" class="toast-message" role="status">
        <svg viewBox="0 0 20 20" aria-hidden="true"><circle cx="10" cy="10" r="8" fill="none" stroke="currentColor" stroke-width="1.5" /><path d="m6.5 10 2.2 2.2 4.8-4.8" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round" /></svg>
        {{ notice }}
      </div>
    </Transition>
  </div>
</template>
