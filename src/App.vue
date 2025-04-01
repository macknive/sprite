<template>
  <div class="container">
    <div
      class="background"
      :style="{
        backgroundImage: `url(${bg})`,
      }"
    >
      <div
        ref="hero"
        class="character"
        :style="{
          left: `${position.x}px`,
          top: `${position.y}px`,
          backgroundImage: `url(${heroSprite})`,
          backgroundPosition: `-${frame * spriteWidth}px -${
            direction * spriteHeight
          }px`,
        }"
      >
        <div
          class="shadow"
          :style="{
            backgroundImage: `url(${shadow})`,
          }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue"
import heroSprite from "@/assets/sprites/hero.png"
import shadow from "@/assets/sprites/shadow.png"
import bg from "@/assets/sprites/bg.webp"

const hero = ref(null)
const position = ref({ x: 500, y: 500 })
const frame = ref(0)
const direction = ref(0)

const speed = 10
const spriteWidth = 32
const spriteHeight = 32
const totalFrames = 3

let moving = false
let animationFrame

const moveHero = (e) => {
  if (moving) return // Prevent duplicate calls
  moving = true

  switch (e.key) {
    case "ArrowUp":
      direction.value = 2 // Third row (Up)
      position.value.y -= speed
      break
    case "ArrowDown":
      direction.value = 0 // First row (Down)
      position.value.y += speed
      break
    case "ArrowLeft":
      direction.value = 3 // Second row (Left)
      position.value.x -= speed
      break
    case "ArrowRight":
      direction.value = 1 // Fourth row (Right)
      position.value.x += speed
      break
    default:
      moving = false
      return
  }

  // Cycle animation frames
  frame.value = (frame.value + 1) % totalFrames

  // Request next animation frame
  animationFrame = requestAnimationFrame(() => {
    moving = false
  })
}

const stopHero = () => {
  cancelAnimationFrame(animationFrame)
  moving = false
}

onMounted(() => {
  window.addEventListener("keydown", moveHero)
  window.addEventListener("keyup", stopHero)
})

onUnmounted(() => {
  window.removeEventListener("keydown", moveHero)
  window.removeEventListener("keyup", stopHero)
})
</script>

<style scoped>
.background {
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center;
  position: absolute;
  z-index: 0;
}
.character {
  position: absolute;
  width: 32px;
  height: 32px;
  transform: scale(5);
  image-rendering: pixelated;
  z-index: 2;
}

.shadow {
  position: relative;
  width: 32px;
  height: 32px;
  z-index: 1;
}
</style>
