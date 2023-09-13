<script setup>
import { nextTick, ref } from 'vue'

const goal = ref([
  {
    opo: 'Web developer',
    img: new URL('@/assets/img/davy.jpg', import.meta.url).href
  },
  {
    opo: 'Network & security',
    img: new URL('@/assets/img/serge.png', import.meta.url).href
  },
  {
    opo: 'Software & AI',
    img: new URL('@/assets/img/kristien.png', import.meta.url).href
  },
  {
    opo: 'Telecom',
    img: new URL('@/assets/img/dirk.png', import.meta.url).href
  },
  {
    opo: 'IoT',
    img: new URL('@/assets/img/mario.png', import.meta.url).href
  }
])
const imgs = goal.value
  .map((g) => ({ img: g.img, rand: Math.random() }))
  .sort((a, b) => (a.rand > b.rand ? 1 : 0))
  .map((g) => g.img)
const opos = goal.value
  .map((g) => ({ opo: g.opo, rand: Math.random() }))
  .sort((a, b) => (a.rand > b.rand ? 1 : 0))
  .map((g) => g.opo)
const pegs = ref(goal.value.map((g, i) => ({ opo: opos[i], img: imgs[i] })))

const history = ref([])
const draggingIndex = ref(null)
const type = ref(null)
const gameOver = ref(false)

function check() {
  const imgs = pegs.value.filter(
    (p) => p.img === goal.value.find((g) => g.opo === p.opo)?.img
  ).length
  const bulls = pegs.value.filter((p, i) => p.opo === goal.value[i].opo).length

  if (imgs === 5 && bulls === 5) {
    gameOver.value = true
    return
  }

  history.value.push({ pegs: JSON.parse(JSON.stringify(pegs.value)), bulls, imgs })
  nextTick(() => scrollTo(0, document.body.scrollHeight))
}

function end(i, e) {
  const t = type.value
  const target = e.currentTarget.tagName
  e.currentTarget.classList.remove('over')
  type.value = null
  if (t === 'opo') {
    pegs.value[draggingIndex.value] = pegs.value.splice(i, 1, pegs.value[draggingIndex.value])[0]
  }
  if (t === 'img' && target === 'FIGURE') {
    const img = pegs.value[i].img
    pegs.value[i].img = pegs.value[draggingIndex.value].img
    pegs.value[draggingIndex.value].img = img
  } else e.preventDefault()
}

function start(p, t, e) {
  type.value = t
  e.stopPropagation()
  draggingIndex.value = p
}

function leave(e) {
  e.currentTarget.classList.remove('over')
}
function over(e) {
  const t = type.value
  const target = e.currentTarget.tagName
  if (t === 'opo' && target === 'DIV') {
    e.currentTarget.classList.add('over')
    e.preventDefault()
    return true
  }
  if (t === 'img' && target === 'FIGURE') {
    e.currentTarget.classList.add('over')
    e.preventDefault()
    return true
  }
}
</script>

<template>
  <h1>EOICTMind</h1>
  <div class="sidebar-layout">
    <section>
      <h2>Instructies</h2>
      <ul>
        <li>Sleep de docenten naar hun juiste vak</li>
        <li>Sleep de vakken in de juiste volgorde</li>
        <li>📍 : aantal juiste posities</li>
        <li>🧑‍🏫 : aantal correcte vak/docent combinaties</li>
      </ul>
    </section>
    <section>
      <h2>Kraak de code!</h2>
      <div class="history">
        <div v-for="(h, i) in history" :key="i">
          <div class="slots">
            <div v-for="p in h.pegs" :key="p.opo" class="card">
              <figure>
                <img :src="p.img" alt="" draggable="false" />
              </figure>
              <p>{{ p.opo }}</p>
            </div>
          </div>
          <div class="result">
            <p>📍{{ h.bulls }}</p>
            <p>🧑‍🏫 {{ h.imgs }}</p>
          </div>
        </div>
      </div>
      <div class="slots" :class="{ 'game-over': gameOver }" >
        <div
            v-for="(p, i) in pegs"
            :key="p.opo"
            @drop.prevent="end(i, $event)"
            @dragover="over"
            @dragleave.prevent="leave"
            @dragstart="start(i, 'opo', $event)"
            class="card"
            draggable="true"
            :class="{target: type === 'opo'}"
        >
          <figure @drop.prevent="end(i, $event)" @dragleave.prevent="leave" @dragover="over" :class="{target: type === 'img'}">
            <img :src="p.img" @dragstart="start(i, 'img', $event)" draggable="true" alt="" />
          </figure>

          <p>{{ p.opo }}</p>
        </div>
      </div>
      <div class="center">
        <button :disabled="gameOver" @click="check">check</button>
        <p>Aantal pogingen: {{ history.length }}</p>
      </div>
    </section>
  </div>
</template>

<style>
body {
  user-select:none
}

.sidebar-layout {
  display:flex;
  gap:4em;
  flex-wrap:wrap;
  align-items:flex-start
}

.sidebar-layout > section:first-child {
  flex:0 1 24rem
}

li {
  margin-block:.5em
}

.history .slots {
  //filter:grayscale(1)
}

.over {
  opacity:.5
}

.slots {
  display:flex;
  justify-content:center;
  gap:1em;
  transition:background-color .2s ease-in-out;
  padding:2em;
  max-width:50rem
}

.slots.game-over {
  background-color:#7cfc00
}

.card {
  min-height:8em;
  min-width:4em;
  box-shadow:0 0 1em rgba(0,0,0,0.2);
  border-radius:.2em;
  display:flex;
  flex-direction:column;
  gap:.5em;
  justify-content:space-between;
  align-items:center;
  transition:opacity .2s ease-in-out, background-color .5s ease-in-out;
  background-color:#fff;
  flex:0 0 20%
}

.card > figure {
  width:100%;
  transition:opacity .2s ease-in-out, background-color .5s ease-in-out;
  margin:0;
  padding:1em
}

.card img {
  display:block;
  width:100%;
  aspect-ratio:1;
  object-fit:cover;
  height:auto
}

.card p {
  padding:1em
}

.card p,button {
  font-weight:500;
  font-variant:all-petite-caps;
  letter-spacing:.1em
}

.history > div {
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:.7rem;
  max-width:40rem;
  gap:2em
}

.history .result {
  font-size:2.5em;
  flex:1 1 10em
}

button {
  background:transparent;
  border:.2em solid;
  font:inherit;
  font-weight:700;
  padding:.5em 1em;
  margin-block:2em;
  display:inline-block;
  box-shadow:0 0 .5em rgba(0,0,0,0.8);
  transition:box-shadow .2s ease-in-out
}

button:enabled:hover,button:enabled:focus-visible {
  box-shadow:0 0 .2em rgba(0,0,0,0.8)
}

.center {
  text-align:center
}

.target {
  background-color: #3cb497;
  outline: .1em dashed white;
  outline-offset: -.2em;
}
</style>
