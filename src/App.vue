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

const speed = 2
const spriteWidth = 32
const spriteHeight = 32
const totalFrames = 3

let keysPressed = new Set()
let animationFrame

let lastFrameTime = 0
const frameDelay = 100

const moveHero = (timestamp = 0) => {
  let dx = 0
  let dy = 0

  // Movement logic
  if (keysPressed.has("ArrowUp")) dy -= speed
  if (keysPressed.has("ArrowDown")) dy += speed
  if (keysPressed.has("ArrowLeft")) dx -= speed
  if (keysPressed.has("ArrowRight")) dx += speed

  if (dx === 0 && dy === 0) return // no movement

  // Update position
  position.value.x += dx
  position.value.y += dy

  // Set direction for animation (you can refine this logic)
  if (dx !== 0) {
    // Any left or right input overrides vertical
    direction.value = dx < 0 ? 3 : 1
  } else if (dy !== 0) {
    // Only vertical if no horizontal input
    direction.value = dy < 0 ? 2 : 0
  }

  if (timestamp - lastFrameTime > frameDelay) {
    frame.value = (frame.value + 1) % totalFrames
    lastFrameTime = timestamp
  }

  // Keep moving
  animationFrame = requestAnimationFrame(moveHero)
}

const handleKeyDown = (e) => {
  keysPressed.add(e.key)

  // Start moving if it's not already
  if (!animationFrame) {
    moveHero()
  }
}

const handleKeyUp = (e) => {
  keysPressed.delete(e.key)

  // Stop moving if no keys are held
  if (keysPressed.size === 0) {
    cancelAnimationFrame(animationFrame)
    animationFrame = null
  }
}

onMounted(() => {
  window.addEventListener("keydown", handleKeyDown)
  window.addEventListener("keyup", handleKeyUp)
})

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyDown)
  window.removeEventListener("keyup", handleKeyUp)
  cancelAnimationFrame(animationFrame)
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
