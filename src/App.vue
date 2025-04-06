<template>
  <div class="container">
    <div
      class="background"
      :style="{
        backgroundImage: `url(${bg})`,
      }"
    >
      <div
        v-if="displayMessage"
        class="chat"
        :style="{
          backgroundImage: `url(${chatBubble})`,
          left: `${position.x - 30}px`,
          top: `${position.y - 190}px`,
        }"
      >
        <span class="message">{{ message }}</span>
      </div>
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
    <input
      type="text"
      ref="myInput"
      @keydown.enter="handleEnter"
      placeholder="Type a
    message"
      style="position: absolute; top: 10px; left: 10px; z-index: 3"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from "vue"
import heroSprite from "@/assets/sprites/hero.png"
import shadow from "@/assets/sprites/shadow.png"
import bg from "@/assets/sprites/bg.webp"
import chatBubble from "@/assets/sprites/chatbubble.png"

const hero = ref(null)
const position = ref({ x: 500, y: 500 })
const frame = ref(0)
const direction = ref(0)
const myInput = ref(null)
const message = ref(null)
const displayMessage = ref(false)

const speed = 2
const spriteWidth = 32
const spriteHeight = 32
const totalFrames = 3

let keysPressed = new Set()
let animationFrame

let lastFrameTime = 0
const frameDelay = 100

let lastKeyPressed = null

const moveHero = (timestamp = 0) => {
  let dx = 0
  let dy = 0

  // Movement logic: prioritize the last key pressed if held down
  if (lastKeyPressed === "ArrowUp") dy -= speed
  if (lastKeyPressed === "ArrowDown") dy += speed
  if (lastKeyPressed === "ArrowLeft") dx -= speed
  if (lastKeyPressed === "ArrowRight") dx += speed

  if (keysPressed.has("ArrowUp") && keysPressed.has("ArrowLeft")) {
    dy -= speed
    dx -= speed
  } else if (keysPressed.has("ArrowUp") && keysPressed.has("ArrowRight")) {
    dy -= speed
    dx += speed
  } else if (keysPressed.has("ArrowDown") && keysPressed.has("ArrowLeft")) {
    dy += speed
    dx -= speed
  } else if (keysPressed.has("ArrowDown") && keysPressed.has("ArrowRight")) {
    dy += speed
    dx += speed
  }

  if (dx !== 0 && dy !== 0) {
    const diagonalSpeed = speed / Math.sqrt(2) // Scale by √2 to normalize
    dx = dx > 0 ? diagonalSpeed : -diagonalSpeed
    dy = dy > 0 ? diagonalSpeed : -diagonalSpeed
  }

  // If no movement input, return early
  if (dx === 0 && dy === 0) return

  // Update position
  position.value.x += dx
  position.value.y += dy

  // Set direction for animation
  if (dx !== 0) {
    direction.value = dx < 0 ? 3 : 1
  } else if (dy !== 0) {
    direction.value = dy < 0 ? 2 : 0
  }

  // Frame update logic
  if (timestamp - lastFrameTime > frameDelay) {
    frame.value = (frame.value + 1) % totalFrames
    lastFrameTime = timestamp
  }

  // Keep moving
  animationFrame = requestAnimationFrame(moveHero)
}

const handleKeyDown = (e) => {
  // Add key to the set and update lastKeyPressed
  keysPressed.add(e.key)
  lastKeyPressed = e.key // Update last key pressed

  // Start moving if not already moving
  if (!animationFrame) {
    moveHero()
  }
}

const handleKeyUp = (e) => {
  // Remove the key from the set
  keysPressed.delete(e.key)

  // If the key released is the last key pressed, check the next available key in the set
  if (e.key === lastKeyPressed) {
    // Find the next key in the set to be held down (if any)
    const nextKey = [...keysPressed].pop() // Get the last key in the set, if any

    // Update lastKeyPressed to the new key, or null if no keys are pressed
    lastKeyPressed = nextKey || null
  }

  // Stop movement if no keys are pressed
  if (keysPressed.size === 0) {
    cancelAnimationFrame(animationFrame)
    animationFrame = null
  }
}

const handleEnter = () => {
  message.value = myInput.value.value
  displayMessage.value = true
  setTimeout(() => {
    displayMessage.value = false
  }, 3000)
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
.chat {
  height: 200px;
  width: 200px;
  background-size: cover;
  position: absolute;
}
.message {
  position: absolute;
  top: 30%;
  left: 20%;
}

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
