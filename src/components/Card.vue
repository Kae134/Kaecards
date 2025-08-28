<script setup lang="ts">
  import { ref } from 'vue';
  import Dracolos from "../../public/test_image.jpg"
  import Foil from "../../public/049_foil_holo_sunpillar_2x.webp"

  const mouseX = ref(0);
  const mouseY = ref(0);
  const focus = ref(0);
  const card = ref<HTMLElement | null>(null)
  const round = ref<HTMLElement | null>(null)

  // Nouvelles variables pour les pourcentages de position
  const mouseXPercent = ref(50);
  const mouseYPercent = ref(50);

  const cardMatrix = ref("1,0,0,0, 0,1,0,0, 0,0,1,0, 0,0,0,1")
  const transition = ref(false)


  const getPosition = (e) => {
    if (!card.value) return
    const rect = e.currentTarget.getBoundingClientRect()
    mouseX.value = e.clientX - rect.left
    mouseY.value = e.clientY - rect.top

    // Calculer les pourcentages pour le gradient
    mouseXPercent.value = (mouseX.value / card.value.offsetWidth) * 100
    mouseYPercent.value = (mouseY.value / card.value.offsetHeight) * 100

    console.log(mouseX.value, mouseY.value, mouseXPercent.value, mouseYPercent.value)

    round.value.style.top = mouseY.value - (round.value.offsetWidth / 2) + "px" 
    round.value.style.left = mouseX.value - (round.value.offsetHeight / 2) + "px" 

    const halfX = card.value.offsetWidth / 2
    const halfY = card.value.offsetHeight / 2

    focus.value = (Math.abs(mouseX.value - halfX) + Math.abs(mouseY.value - halfY)) / 220

    console.log(focus.value)

    const smallX = ((mouseX.value - halfX) / 200000)
    const smallY = ((mouseY.value - halfY) / 200000)
    
    cardMatrix.value = `1,0,0,${-smallX}, 0,1,0,${-smallY}, 0,0,1,0, 0,0,0,1`
  }

  const enter = () => {
    transition.value = false
  }

  const leave = (e) => {
    transition.value = true
    if (!card.value) return
    mouseX.value = 0
    mouseY.value = 0
    mouseXPercent.value = 50
    mouseYPercent.value = 50

    cardMatrix.value = "1,0,0,0, 0,1,0,0, 0,0,1,0, 0,0,0,1"
    console.log(cardMatrix.value)
  }

  
</script>

<template>
  <div class="cardReceptor">

    <div ref="card" :style="{ 
      '--cardPosition': cardMatrix , 
      '--transition' : transition ? 'all 0.5s ease' : '',
    }" class="card" @mousemove="getPosition" @mouseenter="enter" @mouseleave="leave">
      <img class="image" :src="Dracolos" alt="">
      <div class="glare" :style="{
        '--mouseXPercent': mouseXPercent + '%',
        '--mouseYPercent': mouseYPercent + '%',
      }"></div>
      <div class="foil" :style="{
        '--scale': 5,
        '--mouseY': mouseY/20 + 'px',
        '--mouseX': mouseX/20 + 'px ',
        '--mouseXPercent': mouseXPercent + '%',
        '--mouseYPercent': mouseYPercent + '%',
        '--space': 5 + 'px',
        '--focus': focus,
        '--foil': `url(${Foil})`,
      }"></div>
      <div class="round" ref="round"></div>
    </div>
  </div>

</template>

<style>
.card {
  position: relative;
  width: 14rem;
  height: 20rem;
  border-radius: 10px;
  overflow: hidden;
  z-index: 10;

  transform: matrix3d(var(--cardPosition));
  transition: var(--transition);

}

.cardReceptor {
  perspective: 1000px;
  width: fit-content;
  height: fit-content;
  margin: 2rem;
}

.glare {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: .6;
  background-image: radial-gradient(
    farthest-corner circle at var(--mouseXPercent) var(--mouseYPercent),
    hsla(0, 0%, 100%, 0.8) 10%,
    hsla(0, 0%, 100%, 0.65) 20%,
    hsla(0, 0%, 0%, 0.5) 90%
  );
  mix-blend-mode: overlay;
  z-index: 10;
  transform: translateZ(1.41px);
  overflow: hidden;
}

.image {
  position: absolute;
  top: 0;
  left: 0;
  width: 14rem;
  height: 20rem;
  object-fit: cover;
  z-index: 1;
}

.foil {
  position: relative;
  top: 0;
  left: 0;
  border-radius: 10px;
  width: 15rem;
  height: 15rem;
  scale: var(--scale);

  opacity: .3;
  background-blend-mode: color-dodge;
  background: transparent;

  background: repeating-linear-gradient(
    133deg,
    rgba(255, 0, 0, 0.75) calc(var(--space) * 1), 
    rgba(255, 225, 0, 0.75) calc(var(--space) * 2), 
    rgba(12, 255, 69, 0.75) calc(var(--space) * 3), 
    rgba(0, 255, 255, 0.75) calc(var(--space) * 4), 
    rgba(4, 0, 255, 0.75) calc(var(--space) * 5), 
    rgba(221, 0, 255, 0.75) calc(var(--space) * 6), 
    rgba(255, 0, 0, 0.75) calc(var(--space) * 7)
  );

  background-size: 200% 200%;
  background-position: var(--mouseX) var(--mouseY);
  filter: brightness(var(--focus));
  z-index: 3;
  transform: translateZ(1px);
}

.foil::after {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;

  background-image: radial-gradient(
    farthest-corner ellipse 
    at var(--mouseXPercent) var(--mouseYPercent), 
    hsl(0, 0%, 100%) 5%, 
      hsla(300, 100%, 11%, 0.6) 40%, 
      hsl(0, 0%, 22%) 120% 
  );

  background-position: center center;
  background-size: 300% 300%;
  filter: brightness(calc((var(--focus)))) contrast(.85) saturate(1.1);
  mix-blend-mode: hard-light;
  transform: translateZ(1.2px);
}

.foil_uv {
  position: absolute;
  top: 0;
  left: 0;
  border-radius: 10px;
  width: 14rem;
  height: 20rem;
  opacity: 1;
  z-index: 10;
}

.round {
  position: absolute;
  top: 0;
  left: 0;
  width: 25rem;
  height: 25rem;
  border-radius: 50%;
  background-color: rgb(213, 213, 213);
  background-position: 0% var(--mouseY), var(--mouseX) var(--mouseY);
  filter: blur(20px);
  opacity: .6;
  z-index: 10;
  mix-blend-mode: soft-light;
}
</style>