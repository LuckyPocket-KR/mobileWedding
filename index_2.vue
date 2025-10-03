<template>
  <div class="invite-root">
    <header class="hero" :style="`background-image: url(${heroImage})`">
      <div class="overlay">
        <img v-if="couple.photo" :src="couple.photo" alt="couple" class="couple-photo" />
        <h1 class="title">{{ couple.nameA }} &amp; {{ couple.nameB }}</h1>
        <p class="subtitle">{{ formattedDate }} · {{ venue.name }}</p>
        <button class="cta" @click="scrollTo('details')">초대장 보기</button>
      </div>
    </header>

    <main>
      <section id="details" class="card details">
        <h2>식 안내</h2>
        <div class="grid">
          <div>
            <h3>일시</h3>
            <p>{{ formattedDate }}<br />{{ formattedTime }}</p>
          </div>
          <div>
            <h3>장소</h3>
            <p>{{ venue.name }}<br />{{ venue.address }}</p>
            <a :href="mapLink" target="_blank" class="map-link">지도 열기</a>
          </div>
        </div>
      </section>

      <section class="card countdown">
        <h2>카운트다운</h2>
        <div class="count">
          <div><strong>{{ remaining.days }}</strong><span>일</span></div>
          <div><strong>{{ remaining.hours }}</strong><span>시간</span></div>
          <div><strong>{{ remaining.minutes }}</strong><span>분</span></div>
          <div><strong>{{ remaining.seconds }}</strong><span>초</span></div>
        </div>
      </section>

      <section class="card gallery">
        <h2>사진첩</h2>
        <div class="carousel">
          <button class="arrow left" @click="prevSlide">‹</button>
          <div class="slides" :style="`transform: translateX(-${currentIndex * 100}%);`">
            <div class="slide" v-for="(img, i) in gallery" :key="i">
              <img :src="img" :alt="`photo-${i}`" />
            </div>
          </div>
          <button class="arrow right" @click="nextSlide">›</button>
        </div>
      </section>

      <section class="card rsvp">
        <h2>참석 여부</h2>
        <form @submit.prevent="submitRsvp">
          <label>
            이름
            <input v-model="rsvp.name" type="text" placeholder="홍길동" required />
          </label>
          <label>
            참석 여부
            <select v-model="rsvp.attend">
              <option value="yes">참석합니다</option>
              <option value="no">불참합니다</option>
            </select>
          </label>
          <label>
            인원
            <input v-model.number="rsvp.guests" type="number" min="1" />
          </label>
          <label>
            메시지 (선택)
            <textarea v-model="rsvp.message" rows="3" placeholder="축하 메시지를 남겨주세요"></textarea>
          </label>
          <div class="actions">
            <button type="submit">전송</button>
            <button type="button" class="secondary" @click="resetRsvp">초기화</button>
          </div>
        </form>
        <p v-if="rsvpSubmitted" class="notice">응답이 정상적으로 제출되었습니다. 감사합니다!</p>
      </section>

      <section class="card music">
        <h2>축가</h2>
        <p>원하시면 음악을 재생해보세요.</p>
        <div class="music-controls">
          <button @click="toggleMusic">{{ isPlaying ? '멈춤' : '재생' }}</button>
          <audio ref="audio" :src="musicSrc" preload="none"></audio>
        </div>
      </section>

      <footer class="card footer">
        <div class="share">
          <button @click="share('copy')">링크 복사</button>
          <button @click="share('kakao')">카카오톡 공유</button>
        </div>
        <p class="credits">Designed with ❤️ · 모바일 청첩장 샘플</p>
      </footer>
    </main>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

// --- 사용자 데이터: 필요에 따라 변경하세요 ---
const couple = reactive({
  nameA: '김연수',
  nameB: '이소영',
  photo: '/images/couple.jpg' // public 폴더에 넣으세요
})

const heroImage = '/images/hero.jpg' // public에 넣을 플레이스홀더
const gallery = [
  '/images/gallery1.jpg',
  '/images/gallery2.jpg',
  '/images/gallery3.jpg'
]
const musicSrc = '/audio/wedding_song.mp3' // 선택사항

const venue = reactive({
  name: '서울 웨딩홀',
  address: '서울특별시 강남구 예시로 123',
  lat: 37.5172,
  lng: 127.0473
})

// 예: 2025-11-15 14:00 형태로 설정
const eventDateISO = '2025-11-15T14:00:00'

// --- RSVP 상태 ---
const rsvp = reactive({ name: '', attend: 'yes', guests: 1, message: '' })
const rsvpSubmitted = ref(false)

// --- 갤러리 슬라이더 ---
const currentIndex = ref(0)
function prevSlide() { currentIndex.value = (currentIndex.value - 1 + gallery.length) % gallery.length }
function nextSlide() { currentIndex.value = (currentIndex.value + 1) % gallery.length }

// --- 카운트다운 ---
const target = new Date(eventDateISO)
const remaining = reactive({ days: 0, hours: 0, minutes: 0, seconds: 0 })
let timer = null
function updateCountdown() {
  const now = new Date()
  let diff = Math.max(0, Math.floor((target - now) / 1000))
  remaining.days = Math.floor(diff / 86400)
  diff %= 86400
  remaining.hours = Math.floor(diff / 3600)
  diff %= 3600
  remaining.minutes = Math.floor(diff / 60)
  remaining.seconds = diff % 60
}

onMounted(() => {
  updateCountdown()
  timer = setInterval(updateCountdown, 1000)
})
onUnmounted(() => { clearInterval(timer) })

const formattedDate = computed(() => target.toLocaleDateString('ko-KR', { year: 'numeric', month: 'long', day: 'numeric' }))
const formattedTime = computed(() => target.toLocaleTimeString('ko-KR', { hour: '2-digit', minute: '2-digit' }))

const mapLink = computed(() => `https://maps.google.com/?q=${venue.lat},${venue.lng}`)

function scrollTo(id) {
  const el = document.getElementById(id)
  if (el) el.scrollIntoView({ behavior: 'smooth' })
}

function submitRsvp() {
  // 실제 서버 연동 대신 로컬에 저장하거나 fetch로 전송하세요.
  console.log('RSVP 제출', { ...rsvp })
  rsvpSubmitted.value = true
  setTimeout(() => (rsvpSubmitted.value = false), 5000)
}
function resetRsvp() { rsvp.name = ''; rsvp.attend = 'yes'; rsvp.guests = 1; rsvp.message = ''; rsvpSubmitted.value = false }

// --- 음악 제어 ---
const audio = ref(null)
const isPlaying = ref(false)
function toggleMusic() {
  if (!audio.value) return
  if (isPlaying.value) {
    audio.value.pause()
    isPlaying.value = false
  } else {
    audio.value.play()
    isPlaying.value = true
  }
}

function share(method) {
  const pageUrl = location.href
  if (method === 'copy') {
    navigator.clipboard?.writeText(pageUrl)
      .then(() => alert('링크가 복사되었습니다'))
      .catch(() => alert('복사에 실패했습니다'))
  } else if (method === 'kakao') {
    alert('카카오톡 공유 연동은 카카오 SDK가 필요합니다. 예시에서는 안내만 표시됩니다.')
  }
}
</script>

<style scoped>
:root { --card-bg: #fff; --accent: #e07a5f; --muted:#666 }
.invite-root { font-family: -apple-system, BlinkMacSystemFont, 'Noto Sans KR', 'Helvetica Neue', Arial; color: #222; background: #f6f6f8 }
.hero { height: 56vh; min-height: 320px; background-size: cover; background-position: center; position: relative; display:flex; align-items:center; justify-content:center }
.hero .overlay { background: linear-gradient(180deg, rgba(0,0,0,0.25), rgba(0,0,0,0.45)); width:100%; height:100%; display:flex; flex-direction:column; align-items:center; justify-content:center; color:#fff; text-align:center; padding:2rem }
.couple-photo { width:120px; height:120px; object-fit:cover; border-radius:999px; border:4px solid rgba(255,255,255,0.9); margin-bottom:0.75rem }
.title { font-size:1.6rem; font-weight:700; margin:0 }
.subtitle { margin:0.25rem 0 1rem; opacity:0.95 }
.cta { background:var(--accent); color:#fff; border:none; padding:0.6rem 1rem; border-radius:999px; cursor:pointer }

main { padding:1rem }
.card { background:var(--card-bg); border-radius:12px; padding:1rem; box-shadow: 0 6px 18px rgba(20,20,30,0.06); margin-bottom:1rem }
.details .grid { display:flex; gap:1rem; flex-wrap:wrap }
.details h3 { margin:0 0 0.25rem }
.map-link { display:inline-block; margin-top:0.5rem; color:var(--accent) }

.countdown .count { display:flex; gap:0.5rem; justify-content:space-between }
.countdown .count > div { flex:1; text-align:center; background:#fafafa; padding:0.6rem; border-radius:8px }
.countdown strong { display:block; font-size:1.2rem }

.gallery .carousel { position:relative; overflow:hidden }
.slides { display:flex; transition:transform 300ms ease }
.slide { min-width:100%; display:flex; align-items:center; justify-content:center }
.slide img { width:100%; height:220px; object-fit:cover; border-radius:8px }
.arrow { position:absolute; top:50%; transform:translateY(-50%); background:rgba(0,0,0,0.35); color:#fff; border:none; width:36px; height:36px; border-radius:999px; cursor:pointer }
.arrow.left { left:8px }
.arrow.right { right:8px }

.rsvp label { display:block; margin-bottom:0.6rem }
.rsvp input, .rsvp select, .rsvp textarea { width:100%; padding:0.5rem; border:1px solid #e6e6e9; border-radius:6px }
.actions { display:flex; gap:0.5rem; margin-top:0.5rem }
.actions button { padding:0.5rem 0.8rem; border-radius:8px; border:none }
.actions .secondary { background:#f4f4f6 }
.notice { margin-top:0.6rem; color:green }

.music-controls { display:flex; gap:0.5rem; align-items:center }
.footer .share { display:flex; gap:0.5rem }
.footer .credits { margin-top:0.5rem; color:var(--muted); font-size:0.9rem }

@media(min-width:720px){ .title { font-size:2.2rem } .slide img { height:320px } }
</style>