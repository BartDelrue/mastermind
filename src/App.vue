<script setup>
import { nextTick, ref } from 'vue'

const goal = ref([
  {
    opo: 'web',
    img: 'https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.specialistspeakers.com%2Fwp-content%2Fuploads%2F2015%2F09%2FTim-Berners-Lee.jpg&f=1&nofb=1&ipt=0b187a2e147d9ed146a7664a1279b517cb6454bcf6cd73709d4283dac8bd8003&ipo=images'
  },
  {
    opo: 'infra',
    img: 'https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse2.mm.bing.net%2Fth%3Fid%3DOIP.M3hTFmUuetXthJXjswBhIwHaKw%26pid%3DApi&f=1&ipt=1ced49fefe754b5a7afee4dd5608814d764051375faa2e3bba57f56b68d7834e&ipo=images'
  },
  {
    opo: 'software',
    img: 'https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.constellationr.com%2Fsites%2Fdefault%2Ffiles%2Fstyles%2Fuser_profile__medium_square%2Fpublic%2Fperson%2FTBLOfficial_FullRes%2520(2).jpg%3Fitok%3DH4CrREce&f=1&nofb=1&ipt=afddd26492fc9ed87b9cc2304d5404117e7a87c2d2583a581d70723fbbe7adaf&ipo=images'
  },
  {
    opo: 'telecom',
    img: 'https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.exec-comms.com%2FPictures%2FBlog2%2Fberners-lee.jpg&f=1&nofb=1&ipt=cbece0e81575910a39b32034f171013101e79131f1f843d45d38b317077a3bce&ipo=images'
  },
  {
    opo: 'iot',
    img: 'https://external-content.duckduckgo.com/iu/?u=https%3A%2F%2Ftse4.mm.bing.net%2Fth%3Fid%3DOIP.13Hg_INME3TXChBxSUM4RQAAAA%26pid%3DApi&f=1&ipt=5d9425b8396ab9274bf0d537dba7e9ef7da913cc19e635ff564956f1bdeef551&ipo=images'
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
  const type = e.dataTransfer.getData('text')
  const target = e.currentTarget.tagName
  e.currentTarget.classList.remove('over')
  if (type === 'opo' && target === 'DIV') {
    pegs.value[draggingIndex.value] = pegs.value.splice(i, 1, pegs.value[draggingIndex.value])[0]
  }
  if (type === 'img' && target === 'FIGURE') {
    const img = pegs.value[i].img
    pegs.value[i].img = pegs.value[draggingIndex.value].img
    pegs.value[draggingIndex.value].img = img
  } else e.preventDefault()
}

function start(p, type, e) {
  e.dataTransfer.setData('text', type)
  e.stopPropagation()
  draggingIndex.value = p
}

function leave(e) {
  e.currentTarget.classList.remove('over')
}
function over(e) {
  const type = e.dataTransfer.getData('text')
  const target = e.currentTarget.tagName

  if (type === 'opo' && target === 'DIV') {
    e.currentTarget.classList.add('over')
    e.preventDefault()
  }
  if (type === 'img' && target === 'FIGURE') {
    e.currentTarget.classList.add('over')
    e.preventDefault()
  }
}
</script>

<template>
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
  <div class="slots" :class="{ 'game-over': gameOver }">
    <div
      v-for="(p, i) in pegs"
      :key="p.opo"
      @drop.prevent="end(i, $event)"
      @dragover="over"
      @dragleave.prevent="leave"
      @dragstart="start(i, 'opo', $event)"
      class="card"
      draggable="true"
    >
      <figure @drop.prevent="end(i, $event)" @dragleave.prevent="leave" @dragover="over">
        <img :src="p.img" @dragstart="start(i, 'img', $event)" draggable="true" alt="" />
      </figure>

      <p>{{ p.opo }}</p>
    </div>
  </div>

  <button :disabled="gameOver" @click="check">check</button>
</template>

<style>
body {
  user-select: none;
}
.history .slots {
  filter: grayscale(1);
}
.over {
  opacity: 0.5;
}

.slots {
  display: flex;
  justify-content: center;
  gap: 1em;
  transition: background-color 0.2s ease-in-out;
  padding: 2em;
}

.slots.game-over {
  background-color: lawngreen;
}

.card {
  min-height: 8em;
  min-width: 4em;
  box-shadow: 0 0 1em rgba(0, 0, 0, 0.2);
  border-radius: 0.2em;
  display: flex;
  flex-direction: column;
  gap: 0.5em;
  justify-content: space-between;
  align-items: center;
  transition: opacity 0.2s ease-in-out;
  background-color: white;
}

.card > figure {
  width: 100%;
  transition: opacity 0.2s ease-in-out;
  margin: 0;
  padding: 1em;
}

.card img {
  display: block;
  width: 6em;
  height: auto;
}

.card p,
button {
  font-weight: 500;
  font-variant: all-petite-caps;
  letter-spacing: 0.1em;
}
.history {
  margin-bottom: 2em;
}
.history > div {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  gap: 2em;
}
.history .result {
  font-size: 2.5em;
}

button {
  background: transparent;
  border: 0.2em solid;
  font: inherit;
  font-weight: bold;
  padding: 0.5em 1em;
  margin-block: 2em;
  display: inline-block;
  box-shadow: 0 0 0.5em rgba(0, 0, 0, 0.8);
  transition: box-shadow 0.2s ease-in-out;
}
button:enabled:hover,
button:enabled:focus-visible {
  box-shadow: 0 0 0.2em rgba(0, 0, 0, 0.8);
}
</style>
