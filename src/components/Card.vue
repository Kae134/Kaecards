<script setup lang="ts">
  import { ref } from 'vue';
  import Dracolos from "../../public/test_image.jpg"
  import Foil from "../../public/049_foil_holo_sunpillar_2x.webp"

  const mouseX = ref(0);
  const mouseY = ref(0);
  const focus = ref(0);
  const card = ref<HTMLElement | null>(null)
  const round = ref<HTMLElement | null>(null)

  const mouseXPercent = ref(50);
  const mouseYPercent = ref(50);

  const transition = ref(false)


  const getPosition = (e:any) => {
    if (!card.value) return
    const rect = card.value.getBoundingClientRect()
    
    const relativeX = e.clientX - rect.left
    const relativeY = e.clientY - rect.top
    
    mouseXPercent.value = (relativeX / rect.width) * 100
    mouseYPercent.value = (relativeY / rect.height) * 100

    const halfX = rect.width / 2  
    const halfY = rect.height / 2 

    focus.value = Math.min((Math.abs(halfX - relativeX) + Math.abs(halfY - relativeY) / (halfX + halfY)) / 100, 1)
  }

  const enter = () => {
    transition.value = false
  }

  const leave = () => {
    transition.value = true
    if (!card.value) return
    mouseX.value = 0
    mouseY.value = 0
    mouseXPercent.value = 50
    mouseYPercent.value = 50
    focus.value = 0
  }

  
</script>

<template>
  <div class="cardReceptor">

    <div ref="card" :style="{ 
      '--degreeY': (50 - mouseXPercent) * .45  + 'deg',
      '--degreeX': ((50 - mouseYPercent) * -1) * .45 + 'deg',
      '--transition' : transition ? 'all 0.5s ease' : '',
    }" class="card" @mousemove="getPosition" @mouseenter="enter" @mouseleave="leave">
      <img class="image" :src="Dracolos" alt="">
      <div class="glare" :style="{
        '--mouseXPercent': mouseXPercent + '%',
        '--mouseYPercent': mouseYPercent + '%',
      }"></div>
      <div class="foil" :style="{
        '--scale': 10,
        '--mouseXPercent': mouseXPercent + '%',
        '--mouseYPercent': mouseYPercent + '%',
        '--space': 5 + '%',
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
  transform: rotateY(var(--degreeY)) rotateX(var(--degreeX));
  transition: var(--transition);
}

.cardReceptor {
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: 1000px;
  width: 30rem;
  height: 30rem;
  margin: 2rem;
}

.glare {
  transform: translateZ(1.41px);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: radial-gradient(
    farthest-corner circle at var(--mouseXPercent) var(--mouseYPercent), 
    rgb(255, 255, 255) 5%, 
    rgba(255, 255, 255, 0.6) 40%, 
    rgb(79, 99, 99) 120% 
  );
  mix-blend-mode: soft-light;
}

.image {
  position: absolute;
  top: 0;
  left: 0;
  width: 14rem;
  height: 20rem;
  object-fit: cover;
}

.foil {
  position: relative;
  top: 0;
  left: 0;
  border-radius: 10px;
  width: 14rem;
  height: 20rem;
  background-blend-mode: color-dodge;
  background-size: 300% 400%;


  background-image: repeating-linear-gradient(
    128deg,
    rgba(255, 0, 0, 0.75) calc(var(--space) * 1), 
    rgba(255, 225, 0, 0.75) calc(var(--space) * 2), 
    rgba(12, 255, 69, 0.75) calc(var(--space) * 3), 
    rgba(0, 255, 255, 0.75) calc(var(--space) * 4), 
    rgba(4, 0, 255, 0.75) calc(var(--space) * 5), 
    rgba(221, 0, 255, 0.75) calc(var(--space) * 6), 
    rgba(255, 0, 0, 0.75) calc(var(--space) * 7)
  );

  mix-blend-mode: color-dodge;
  background-position: var(--mouseXPercent) var(--mouseYPercent);
  filter: brightness(calc(var(--focus) * .3 + .4)) contrast(2.3) saturate(1);;
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
    rgb(255, 255, 255) 10%, 
    rgba(56, 0, 56, 0.6) 35%, 
    rgb(0, 0, 0) 60%
  );

  background-position: center center;
  background-size: 400% 500%;
  filter: brightness(calc((var(--focus)) * .2 + .4)) contrast(.8) saturate(1);
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
  width: 10rem;
  height: 10rem;
  border-radius: 50%;
  background-color: rgb(213, 213, 213);
  background-position: 0% var(--mouseYPercent), var(--mouseXPercent) var(--mouseYPercent);
  filter: blur(20px);
  opacity: .6;
  z-index: 10;
  mix-blend-mode: soft-light;
}
</style>