<template>
  <canvas ref="canvasRef" />
</template>

<script setup>
/**
 * 东方梦无垠鼠标粒子效果复刻
 *
 * 机制：20个粒子各自有不同 speed(0.003~0.022)，
 * 通过 ExpoTrend 插值从屏幕右侧趋近鼠标轨道。
 * "先后顺序"完全来自 speed 差异，没有分阶段逻辑。
 *
 * 唯一非源码值：PRE_FRAMES（模拟游戏加载期间粒子已提前收敛）
 */
import { ref, onMounted, onBeforeUnmount } from 'vue'

const COUNT = 20
const canvasRef = ref(null)
let ctx, offCanvas, offCtx
let W, H, dpr
let mouseX = 0, mouseY = 0
let frameCount = 0
let particles = []
let rafId

/** 全局显隐渐变，speed=0.1（源码 DEFAULT_EXPO_SPEED） */
let timerDisp = 0
const TIMER_DISP_SPEED = 0.1

/**
 * 轨道半径（像素）
 * 源码: 100世界单位 × SC(0.0203) × PPU(≈89)
 * PPU 从实测环大小反推，因 luaSTG-x 3DUI 投影有额外变换
 */
const RADIUS_BASE = 180

function createParticle(i) {
  // speed: 源码 0.002 + 0.001 * i (i=1~20) → 0.003~0.022
  const speed = 0.002 + 0.001 * (i + 1)
  return {
    index: i + 1,           // 1-based，和源码 self.__index 一致
    speed,
    phase: Math.random() * 360,  // 随机初始角度（度数）
    x: 0, y: 0,
    vx: 0, vy: 0,
  }
}

onMounted(() => {
  const cvs = canvasRef.value
  ctx = cvs.getContext('2d')
  offCanvas = document.createElement('canvas')
  offCtx = offCanvas.getContext('2d')

  function resize() {
    dpr = window.devicePixelRatio || 1
    W = window.innerWidth
    H = window.innerHeight
    cvs.width = W * dpr
    cvs.height = H * dpr
    cvs.style.width = W + 'px'
    cvs.style.height = H + 'px'
    offCanvas.width = W * dpr
    offCanvas.height = H * dpr
  }
  resize()

  mouseX = W / 2
  mouseY = H / 2

  /**
   * 初始位置：源码 x=320或747, y=0或480（逻辑坐标854×480）
   * 经3D投影后在视野外数倍，INIT_SCALE 控制偏移距离
   */
  const INIT_SCALE = 32768

  for (let i = 0; i < COUNT; i++) {
    const p = createParticle(i)
    const lx = Math.random() < 0.5 ? 320 : 747
    const ly = Math.random() < 0.5 ? 0 : 480
    p.x = mouseX + (lx / 854 - 0.5) * W * INIT_SCALE
    p.y = mouseY + (ly / 480 - 0.5) * H * INIT_SCALE
    particles.push(p)
  }

  /** 模拟加载时间：粒子提前收敛（游戏里面板创建时就开始计时） */
  const PRE_FRAMES = 300
  for (let f = 0; f < PRE_FRAMES; f++) {
    timerDisp += (1 - timerDisp) * TIMER_DISP_SPEED
    for (const p of particles) {
      const r = RADIUS_BASE * (1 + p.speed * 2)
      const phRad = (f * p.index / 20 + p.phase) * Math.PI / 180
      const tx = mouseX + Math.cos(phRad) * r
      const ty = mouseY - Math.sin(phRad) * r / 3 - (p.index - 10) * 5
      p.x += (tx - p.x) * p.speed
      p.y += (ty - p.y) * p.speed
    }
  }

  function onMouseMove(e) { mouseX = e.clientX; mouseY = e.clientY }
  window.addEventListener('resize', resize)
  window.addEventListener('mousemove', onMouseMove)

  /** 颜色: 源码 water_color = (0.66825, 0.3681, 0.8999) × 255 */
  const CR = 170, CG = 94, CB = 230

  // ── 逻辑更新（固定60FPS，与游戏帧率一致） ──

  function tick() {
    frameCount++
    timerDisp += (1 - timerDisp) * TIMER_DISP_SPEED

    for (const p of particles) {
      /**
       * 轨道目标点
       * r = 100*(1+speed*2)，speed大的粒子轨道更外圈
       * 角速度 = index/20 度/帧，高index转得快
       * Y压缩1/3（椭圆），垂直偏移(index-10)/10分散粒子
       */
      const r = RADIUS_BASE * (1 + p.speed * 2)
      const phRad = (frameCount * p.index / 20 + p.phase) * Math.PI / 180
      const targetX = mouseX + Math.cos(phRad) * r
      const targetY = mouseY - Math.sin(phRad) * r / 3 - (p.index - 10) * 5

      // ExpoTrend: value += (aim - value) * speed
      const prevX = p.x
      const prevY = p.y
      p.x += (targetX - p.x) * p.speed
      p.y += (targetY - p.y) * p.speed
      p.vx = p.x - prevX
      p.vy = p.y - prevY
    }
  }

  // ── 渲染 ──

  function draw() {
    offCtx.clearRect(0, 0, W * dpr, H * dpr)
    offCtx.save()
    offCtx.scale(dpr, dpr)
    offCtx.globalCompositeOperation = 'lighter'  // mul+add
    offCtx.lineCap = 'round'

    for (const p of particles) {
      if (p.x < -100 || p.x > W + 100 || p.y < -100 || p.y > H + 100) continue

      const vel = Math.sqrt(p.vx * p.vx + p.vy * p.vy)
      const angle = Math.atan2(p.vy, p.vx)
      /**
       * alpha = speed/maxSpeed * timer_disp
       * maxSpeed = 0.003 + 20*0.002 = 0.043
       */
      const alpha = (p.speed / 0.043) * timerDisp

      /** 线条尺寸: 源码 scaleX=hypot/20+0.002, scaleY=0.001 × 3D投影 */
      const len = 5 + vel * 0.8
      const w = 1.5

      offCtx.save()
      offCtx.translate(p.x, p.y)
      offCtx.rotate(angle)

      offCtx.globalAlpha = alpha
      offCtx.strokeStyle = `rgb(${CR}, ${CG}, ${CB})`
      offCtx.lineWidth = w
      offCtx.beginPath()
      offCtx.moveTo(-len / 2, 0)
      offCtx.lineTo(len / 2, 0)
      offCtx.stroke()

      // 白色芯线：模拟 mul+add 叠加的高亮中心
      offCtx.globalAlpha = alpha * 0.4
      offCtx.strokeStyle = `rgb(230, 220, 255)`
      offCtx.lineWidth = w * 0.3
      offCtx.beginPath()
      offCtx.moveTo(-len / 2, 0)
      offCtx.lineTo(len / 2, 0)
      offCtx.stroke()

      offCtx.restore()
    }

    offCtx.restore()

    ctx.clearRect(0, 0, W * dpr, H * dpr)
    ctx.save()
    ctx.scale(dpr, dpr)
    ctx.globalCompositeOperation = 'lighter'

    // 辉光层：近似 mul+add 的视觉光晕
    ctx.filter = 'blur(4px)'
    ctx.globalAlpha = 0.5
    ctx.drawImage(offCanvas, 0, 0, W, H)

    ctx.filter = 'none'
    ctx.globalAlpha = 1
    ctx.drawImage(offCanvas, 0, 0, W, H)

    ctx.restore()
  }

  // ── 固定时间步长（游戏锁60FPS，浏览器可能120/144Hz） ──

  const FRAME_MS = 1000 / 60
  let lastTime = 0
  let accumulator = 0

  function loop(now) {
    if (!lastTime) lastTime = now
    accumulator += now - lastTime
    lastTime = now

    while (accumulator >= FRAME_MS) {
      tick()
      accumulator -= FRAME_MS
    }

    draw()
    rafId = requestAnimationFrame(loop)
  }

  rafId = requestAnimationFrame(loop)

  onBeforeUnmount(() => {
    cancelAnimationFrame(rafId)
    window.removeEventListener('resize', resize)
    window.removeEventListener('mousemove', onMouseMove)
  })
})
</script>

<style scoped>
canvas {
  position: fixed;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 9999;
}
</style>