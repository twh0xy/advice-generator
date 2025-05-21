<script setup lang="ts">
import { ref, onMounted, nextTick, onScopeDispose } from 'vue'
import { gsap } from 'gsap'

const API_URL = 'https://api.adviceslip.com/advice'
const MAX_RETRIES = 5
const ANIMATION = {
  fadeOut: { opacity: 0, y: 20, duration: 0.3 },
  fadeIn: { opacity: 1, y: 0, duration: 0.5, ease: 'power2.out' },
  spin: {
    rotation: 360,
    duration: 0.9,
    ease: 'power2.inOut',
    repeat: -1,
  },
}

const adviceId = ref<number | null>(null)
const adviceText = ref('')
const isLoading = ref(false)
const retryCount = ref(0)
const diceAnimation = ref<GSAPTween | null>(null)

const animateElement = (
  selector: string,
  animation: gsap.TweenVars,
  isFromTo: boolean = false,
  fromProps?: gsap.TweenVars,
) => {
  return isFromTo ? gsap.fromTo(selector, fromProps!, animation) : gsap.to(selector, animation)
}

const startDiceSpin = () => {
  if (diceAnimation.value) {
    diceAnimation.value.kill()
    gsap.set('.advice__card--refresh svg', { rotation: 0 })
  }

  diceAnimation.value = animateElement('.advice__card--refresh svg', ANIMATION.spin, true, {
    rotation: 0,
  })
}

async function fetchRandomAdvice() {
  try {
    const response = await fetch(API_URL, { cache: 'no-store' })
    if (!response.ok) throw new Error('Erro na API')
    return await response.json()
  } catch (error) {
    if (retryCount.value++ < MAX_RETRIES) return fetchRandomAdvice()
    throw new Error('Tentativas máximas atingidas')
  }
}

async function updateAdvice() {
  isLoading.value = true
  retryCount.value = 0
  startDiceSpin()

  try {
    if (adviceId.value !== null) {
      await animateElement('.advice__card--content', ANIMATION.fadeOut)
      await new Promise((resolve) => setTimeout(resolve, 300))
    }

    const { slip } = await fetchRandomAdvice()
    if (!slip?.id || !slip?.advice) throw new Error('Formato inválido')

    adviceId.value = slip.id
    adviceText.value = slip.advice
    await nextTick()

    animateElement('.advice__card--content', ANIMATION.fadeIn, true, { opacity: 0, y: -20 })
  } finally {
    diceAnimation.value?.kill()
    gsap.set('.advice__card--refresh svg', { rotation: 0 })
    isLoading.value = false
  }
}

async function handleRefresh() {
  if (isLoading.value) return
  startDiceSpin()
  await updateAdvice()
}

onScopeDispose(() => {
  diceAnimation.value?.kill()
})

onMounted(updateAdvice)
</script>

<template>
  <div class="advice__card">
    <div class="advice__card--hcontent">
      <p class="advice__card--tag">
        ADVICE <span class="advice__card--id">#{{ adviceId ?? '...' }}</span>
      </p>
      <p class="advice__card--content">"{{ adviceText }}"</p>
      <div class="advice__card--details-st">
        <div class="advice__card--risk"></div>
        <p class="advice__card--pause">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="32"
            height="32"
            viewBox="0 0 24 24"
            fill="currentColor"
            class="icon icon-tabler icons-tabler-filled icon-tabler-player-pause"
          >
            <path stroke="none" d="M0 0h24v24H0z" fill="none" />
            <path
              d="M9 4h-2a2 2 0 0 0 -2 2v12a2 2 0 0 0 2 2h2a2 2 0 0 0 2 -2v-12a2 2 0 0 0 -2 -2z"
            />
            <path
              d="M17 4h-2a2 2 0 0 0 -2 2v12a2 2 0 0 0 2 2h2a2 2 0 0 0 2 -2v-12a2 2 0 0 0 -2 -2z"
            />
          </svg>
        </p>
        <div class="advice__card--risk"></div>
      </div>
    </div>
    <div class="advice__card--ctoggle">
      <button class="advice__card--refresh" @click="handleRefresh" :disabled="isLoading">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="28"
          height="28"
          viewBox="0 0 24 24"
          fill="currentColor"
          class="icon icon-tabler icons-tabler-filled icon-tabler-dice"
        >
          <path stroke="none" d="M0 0h24v24H0z" fill="none" />
          <path
            d="M18.333 2c1.96 0 3.56 1.537 3.662 3.472l.005 .195v12.666c0 1.96 -1.537 3.56 -3.472 3.662l-.195 .005h-12.666a3.667 3.667 0 0 1 -3.662 -3.472l-.005 -.195v-12.666c0 -1.96 1.537 -3.56 3.472 -3.662l.195 -.005h12.666zm-2.833 12a1.5 1.5 0 1 0 0 3a1.5 1.5 0 0 0 0 -3zm-7 0a1.5 1.5 0 1 0 0 3a1.5 1.5 0 0 0 0 -3zm0 -7a1.5 1.5 0 1 0 0 3a1.5 1.5 0 0 0 0 -3zm7 0a1.5 1.5 0 1 0 0 3a1.5 1.5 0 0 0 0 -3z"
          />
        </svg>
      </button>
    </div>
  </div>
</template>

<style scoped>
.advice__card {
  width: 600px;
  margin: 0 auto;
  box-shadow: rgba(17, 12, 46, 0.15) 0px 48px 100px 0px;

  padding-top: 50px;
  padding-bottom: 20px;
  padding-left: 20px;
  padding-right: 20px;

  border-radius: 12px;
  background-color: hsl(218, 20%, 24%);

  & .advice__card--hcontent {
    width: 500px;
    margin: 0 auto;
    display: flex;
    gap: 20px;
    text-align: center;
    flex-direction: column;
    padding-bottom: 40px;

    & .advice__card--tag,
    .advice__card--id {
      color: #53ffa9;
      font-weight: 700;
      letter-spacing: 6px;
    }

    & .advice__card--content {
      color: hsl(196, 38%, 86%);
      font-size: 32px;
      font-weight: 700;
      line-height: 46px;
    }

    & .advice__card--details-st {
      display: flex;
      gap: 10px;
      align-items: center;
      justify-content: space-between;

      & .advice__card--risk {
        background-color: hsl(216, 19%, 38%);
        height: 1px;
        width: 215px;
      }
    }
  }

  & .advice__card--ctoggle {
    & .advice__card--refresh {
      cursor: pointer;
      position: fixed;
      top: 93%;
      left: 45%;
      border: 1px solid hsl(150, 56%, 49%);
      border-radius: 60px;
      padding: 10px 14px;
      background-color: hsl(151, 100%, 66%);
      box-shadow:
        rgba(50, 50, 93, 0.25) 0px 2px 5px -1px,
        rgba(0, 0, 0, 0.3) 0px 1px 3px -1px;
      z-index: 9999;
      transition: 0.2s ease-in-out;

      &:hover {
        background-color: hsl(151, 71%, 51%);
      }
    }
  }
}
</style>
