<template>
  <div class="canvas-container">
    <canvas ref="canvas" :width="canvasSize" :height="canvasSize"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvas = ref(null)
const canvasSize = 500
let ctx, animationId

const opts = {
  particles: 200,
  particleBaseSize: 4,
  particleAddedSize: 1,
  particleMaxSize: 5,
  particleBaseLight: 5,
  particleAddedLight: 30,
  particleBaseBaseAngSpeed: .001,
  particleAddedBaseAngSpeed: .001,
  particleBaseVariedAngSpeed: .0005,
  particleAddedVariedAngSpeed: .0005,
  sourceBaseSize: 3,
  sourceAddedSize: 3,
  sourceBaseAngSpeed: -.01,
  sourceVariedAngSpeed: .005,
  sourceBaseDist: 130,
  sourceVariedDist: 50,
  particleTemplateColor: 'hsla(hue,80%,light%,alp)',
  repaintColor: 'rgba(0,0,0,.2)',
  enableTrails: false
}

const util = {
  square: x => x*x,
  tau: 6.2831853071795864769252867665590057683943
}

let particles = []
let source
let tick = 0

class Particle {
  constructor() {
    this.dist = (Math.sqrt(Math.random())) * canvasSize / 2
    this.rad = Math.random() * util.tau
    this.baseAngSpeed = opts.particleBaseBaseAngSpeed + opts.particleAddedBaseAngSpeed * Math.random()
    this.variedAngSpeed = opts.particleBaseVariedAngSpeed + opts.particleAddedVariedAngSpeed * Math.random()
    this.size = opts.particleBaseSize + opts.particleAddedSize * Math.random()
  }

  step() {
    const angSpeed = this.baseAngSpeed + this.variedAngSpeed * Math.sin(this.rad * 7 + tick / 100)
    this.rad += angSpeed

    const x = this.dist * Math.cos(this.rad)
    const y = this.dist * Math.sin(this.rad)

    const squareDist = util.square(x - source.x) + util.square(y - source.y)
    const sizeProp = Math.pow(canvasSize, 1/2) / Math.pow(squareDist, 1/2)
    const color = opts.particleTemplateColor
      .replace('hue', this.rad / util.tau * 360 + tick)
      .replace('light', opts.particleBaseLight + sizeProp * opts.particleAddedLight)
      .replace('alp', .8)

    ctx.fillStyle = color
    ctx.beginPath()
    ctx.arc(x, y, Math.min(this.size * sizeProp, opts.particleMaxSize), 0, util.tau)
    ctx.fill()
  }
}

class Source {
  constructor() {
    this.x = 0
    this.y = 0
    this.rad = Math.random() * util.tau
    this.mouseControlled = false
  }

  step() {
    if (!this.mouseControlled) {
      const angSpeed = opts.sourceBaseAngSpeed + Math.sin(this.rad * 6 + tick / 100) * opts.sourceVariedAngSpeed
      this.rad += angSpeed

      const dist = opts.sourceBaseDist + Math.sin(this.rad * 5 + tick / 100) * opts.sourceVariedDist

      this.x = dist * Math.cos(this.rad)
      this.y = dist * Math.sin(this.rad)
    }

    ctx.fillStyle = 'white'
    ctx.beginPath()
    ctx.arc(this.x, this.y, 1, 0, util.tau)
    ctx.fill()
  }
}

function anim() {
  animationId = window.requestAnimationFrame(anim)

  ++tick

  if (!opts.enableTrails)
    ctx.globalCompositeOperation = 'source-over'

  ctx.fillStyle = opts.repaintColor
  ctx.fillRect(0, 0, canvasSize, canvasSize)

  ctx.globalCompositeOperation = 'lighter'

  if (particles.length < opts.particles)
    particles.push(new Particle())

  ctx.translate(canvasSize/2, canvasSize/2)

  source.step()

  particles.forEach(particle => particle.step())
  ctx.translate(-canvasSize/2, -canvasSize/2)
}

onMounted(() => {
  ctx = canvas.value.getContext('2d')
  source = new Source()

  ctx.fillStyle = '#222'
  ctx.fillRect(0, 0, canvasSize, canvasSize)
  anim()

  canvas.value.addEventListener('mousemove', handleMouseMove)
  canvas.value.addEventListener('mouseleave', handleMouseLeave)
})

onUnmounted(() => {
  if (animationId) {
    window.cancelAnimationFrame(animationId)
  }
  canvas.value.removeEventListener('mousemove', handleMouseMove)
  canvas.value.removeEventListener('mouseleave', handleMouseLeave)
})

function handleMouseMove(e) {
  const bbox = canvas.value.getBoundingClientRect()
  source.x = e.clientX - bbox.left - canvasSize/2
  source.y = e.clientY - bbox.top - canvasSize/2
  source.mouseControlled = true
}

function handleMouseLeave(e) {
  const bbox = canvas.value.getBoundingClientRect()
  source.x = e.clientX - bbox.left - canvasSize/2
  source.y = e.clientY - bbox.top - canvasSize/2
  source.rad = Math.atan(source.y / source.x)
  if (source.x < 0)
    source.rad += Math.PI
  source.mouseControlled = false
}
</script>

<style scoped>
:global(body) {
  background-color: #000;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

.canvas-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
}

canvas {
  width: 500px;  /* 确保与 canvasSize 变量一致 */
  height: 500px; /* 确保与 canvasSize 变量一致 */
  box-shadow: 0 0 2px #111;
  border-radius: 50%;
}
</style>