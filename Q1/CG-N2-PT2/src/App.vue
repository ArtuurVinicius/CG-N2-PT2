<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

// Canvas and camera properties
const canvasRef = ref(null)
const ctx = ref(null)

// Camera system
const CAMERA_WIDTH = 800
const CAMERA_HEIGHT = 600
const VIRTUAL_WIDTH = CAMERA_WIDTH * 2
const VIRTUAL_HEIGHT = CAMERA_HEIGHT * 2

const camera = ref({
  x: 0,
  y: 0,
  targetX: 0,
  targetY: 0,
  smoothing: 0.1
})

// Speedy Gonzales character
const speedy = ref({
  x: VIRTUAL_WIDTH / 2,
  y: VIRTUAL_HEIGHT / 2,
  vx: 0,
  vy: 0,
  speed: 2,
  direction: 0,
  width: 40,
  height: 60,
  targetDirection: 0,
  acceleration: 0.1,
  maxSpeed: 4,
  animationFrame: 0
})

// Input handling
const keys = ref({
  KeyW: false, KeyA: false, KeyS: false, KeyD: false,
  ArrowUp: false, ArrowLeft: false, ArrowDown: false, ArrowRight: false
})

// Light system (Phong illumination)
const light = ref({
  direction: { x: -0.5, y: -0.7, z: -0.5 },
  color: { r: 255, g: 245, b: 200 },
  intensity: 1.2,
  ambient: 0.3,
  diffuse: 0.7,
  specular: 0.4
})

// Environment obstacles and elements
const obstacles = ref([
  { x: 200, y: 150, width: 100, height: 100, type: 'wall' },
  { x: 600, y: 300, width: 80, height: 120, type: 'wall' },
  { x: 1000, y: 200, width: 150, height: 60, type: 'platform' },
  { x: 300, y: 800, width: 200, height: 100, type: 'wall' },
  { x: 1200, y: 600, width: 100, height: 100, type: 'wall' }
])

// Animation state
let animationId = null
let lastTime = 0

// Utility functions for affine transformations
function rotatePoint(x, y, cx, cy, angle) {
  const cos = Math.cos(angle)
  const sin = Math.sin(angle)
  const dx = x - cx
  const dy = y - cy
  return {
    x: cx + dx * cos - dy * sin,
    y: cy + dx * sin + dy * cos
  }
}

function applyPhongLighting(baseColor, normal, position) {
  // Normalize light direction
  const lightDir = light.value.direction
  const lightMag = Math.sqrt(lightDir.x * lightDir.x + lightDir.y * lightDir.y + lightDir.z * lightDir.z)
  const normalizedLight = {
    x: lightDir.x / lightMag,
    y: lightDir.y / lightMag,
    z: lightDir.z / lightMag
  }

  // Calculate ambient component
  const ambient = {
    r: baseColor.r * light.value.ambient,
    g: baseColor.g * light.value.ambient,
    b: baseColor.b * light.value.ambient
  }

  // Calculate diffuse component
  const dotProduct = Math.max(0, normal.x * normalizedLight.x + normal.y * normalizedLight.y + normal.z * normalizedLight.z)
  const diffuse = {
    r: baseColor.r * light.value.diffuse * dotProduct,
    g: baseColor.g * light.value.diffuse * dotProduct,
    b: baseColor.b * light.value.diffuse * dotProduct
  }

  // Calculate specular component (simplified for 2D)
  const viewDir = { x: 0, y: 0, z: 1 }
  const reflectDir = {
    x: 2 * dotProduct * normal.x - normalizedLight.x,
    y: 2 * dotProduct * normal.y - normalizedLight.y,
    z: 2 * dotProduct * normal.z - normalizedLight.z
  }
  const specDot = Math.max(0, viewDir.x * reflectDir.x + viewDir.y * reflectDir.y + viewDir.z * reflectDir.z)
  const specular = {
    r: light.value.color.r * light.value.specular * Math.pow(specDot, 32),
    g: light.value.color.g * light.value.specular * Math.pow(specDot, 32),
    b: light.value.color.b * light.value.specular * Math.pow(specDot, 32)
  }

  // Combine all components
  const final = {
    r: Math.min(255, (ambient.r + diffuse.r + specular.r) * light.value.intensity),
    g: Math.min(255, (ambient.g + diffuse.g + specular.g) * light.value.intensity),
    b: Math.min(255, (ambient.b + diffuse.b + specular.b) * light.value.intensity)
  }

  return `rgb(${Math.floor(final.r)}, ${Math.floor(final.g)}, ${Math.floor(final.b)})`
}

// Character animation and AI
function updateSpeedyMovement(deltaTime) {
  const speedyRef = speedy.value
  
  // Random direction changes every few seconds
  if (Math.random() < 0.02) {
    speedyRef.targetDirection = Math.random() * Math.PI * 2
  }

  // Obstacle avoidance
  const futureX = speedyRef.x + Math.cos(speedyRef.direction) * speedyRef.speed * 10
  const futureY = speedyRef.y + Math.sin(speedyRef.direction) * speedyRef.speed * 10
  
  for (const obstacle of obstacles.value) {
    if (futureX < obstacle.x + obstacle.width + 50 &&
        futureX + speedyRef.width > obstacle.x - 50 &&
        futureY < obstacle.y + obstacle.height + 50 &&
        futureY + speedyRef.height > obstacle.y - 50) {
      // Turn away from obstacle
      speedyRef.targetDirection = Math.atan2(speedyRef.y - obstacle.y, speedyRef.x - obstacle.x)
      break
    }
  }

  // Boundary avoidance
  const margin = 100
  if (speedyRef.x < margin) speedyRef.targetDirection = 0
  if (speedyRef.x > VIRTUAL_WIDTH - margin) speedyRef.targetDirection = Math.PI
  if (speedyRef.y < margin) speedyRef.targetDirection = Math.PI / 2
  if (speedyRef.y > VIRTUAL_HEIGHT - margin) speedyRef.targetDirection = -Math.PI / 2

  // Smooth direction transition
  let angleDiff = speedyRef.targetDirection - speedyRef.direction
  if (angleDiff > Math.PI) angleDiff -= Math.PI * 2
  if (angleDiff < -Math.PI) angleDiff += Math.PI * 2
  speedyRef.direction += angleDiff * 0.05

  // Update velocity with acceleration
  const targetVx = Math.cos(speedyRef.direction) * speedyRef.speed
  const targetVy = Math.sin(speedyRef.direction) * speedyRef.speed
  
  speedyRef.vx += (targetVx - speedyRef.vx) * speedyRef.acceleration
  speedyRef.vy += (targetVy - speedyRef.vy) * speedyRef.acceleration

  // Apply movement
  speedyRef.x += speedyRef.vx
  speedyRef.y += speedyRef.vy

  // Animation frame for sprite animation
  speedyRef.animationFrame += 0.2
}

// Camera movement
function updateCamera() {
  const cameraRef = camera.value
  
  // Keyboard camera controls
  const cameraSpeed = 5
  if (keys.value.KeyW || keys.value.ArrowUp) cameraRef.targetY -= cameraSpeed
  if (keys.value.KeyS || keys.value.ArrowDown) cameraRef.targetY += cameraSpeed
  if (keys.value.KeyA || keys.value.ArrowLeft) cameraRef.targetX -= cameraSpeed
  if (keys.value.KeyD || keys.value.ArrowRight) cameraRef.targetX += cameraSpeed

  // Constrain camera to virtual world bounds
  cameraRef.targetX = Math.max(0, Math.min(VIRTUAL_WIDTH - CAMERA_WIDTH, cameraRef.targetX))
  cameraRef.targetY = Math.max(0, Math.min(VIRTUAL_HEIGHT - CAMERA_HEIGHT, cameraRef.targetY))

  // Smooth camera movement
  cameraRef.x += (cameraRef.targetX - cameraRef.x) * cameraRef.smoothing
  cameraRef.y += (cameraRef.targetY - cameraRef.y) * cameraRef.smoothing
}

// Rendering functions
function drawBackground() {
  const canvas = canvasRef.value
  const context = ctx.value
  
  // Create gradient background
  const gradient = context.createLinearGradient(0, 0, 0, canvas.height)
  gradient.addColorStop(0, '#87CEEB')
  gradient.addColorStop(0.7, '#98FB98')
  gradient.addColorStop(1, '#DEB887')
  
  context.fillStyle = gradient
  context.fillRect(0, 0, canvas.width, canvas.height)

  // Draw grid pattern for reference
  context.strokeStyle = 'rgba(255, 255, 255, 0.1)'
  context.lineWidth = 1
  const gridSize = 50
  
  for (let x = (-camera.value.x % gridSize); x < canvas.width; x += gridSize) {
    context.beginPath()
    context.moveTo(x, 0)
    context.lineTo(x, canvas.height)
    context.stroke()
  }
  
  for (let y = (-camera.value.y % gridSize); y < canvas.height; y += gridSize) {
    context.beginPath()
    context.moveTo(0, y)
    context.lineTo(canvas.width, y)
    context.stroke()
  }
}

function drawObstacles() {
  const context = ctx.value
  const cameraRef = camera.value
  
  context.save()
  
  for (const obstacle of obstacles.value) {
    const screenX = obstacle.x - cameraRef.x
    const screenY = obstacle.y - cameraRef.y
    
    // Only draw if visible on screen
    if (screenX + obstacle.width > 0 && screenX < CAMERA_WIDTH &&
        screenY + obstacle.height > 0 && screenY < CAMERA_HEIGHT) {
      
      // Apply Phong lighting to obstacles
      const normal = { x: 0, y: 0, z: 1 }
      const baseColor = obstacle.type === 'wall' ? { r: 139, g: 69, b: 19 } : { r: 105, g: 105, b: 105 }
      const litColor = applyPhongLighting(baseColor, normal, { x: obstacle.x, y: obstacle.y })
      
      context.fillStyle = litColor
      context.fillRect(screenX, screenY, obstacle.width, obstacle.height)
      
      // Add depth with shadow
      context.fillStyle = 'rgba(0, 0, 0, 0.3)'
      context.fillRect(screenX + 5, screenY + 5, obstacle.width, obstacle.height)
    }
  }
  
  context.restore()
}

function drawSpeedyGonzales() {
  const context = ctx.value
  const speedyRef = speedy.value
  const cameraRef = camera.value
  
  const screenX = speedyRef.x - cameraRef.x
  const screenY = speedyRef.y - cameraRef.y
  
  // Only draw if visible on screen
  if (screenX + speedyRef.width > -50 && screenX < CAMERA_WIDTH + 50 &&
      screenY + speedyRef.height > -50 && screenY < CAMERA_HEIGHT + 50) {
    
    context.save()
    context.translate(screenX + speedyRef.width / 2, screenY + speedyRef.height / 2)
    context.rotate(speedyRef.direction)
    
    // Apply lighting to character
    const normal = { x: 0, y: 0, z: 1 }
    
    // Draw Speedy Gonzales body (yellow mouse)
    const bodyColor = { r: 255, g: 255, b: 0 }
    const litBodyColor = applyPhongLighting(bodyColor, normal, { x: speedyRef.x, y: speedyRef.y })
    context.fillStyle = litBodyColor
    context.fillRect(-speedyRef.width / 2, -speedyRef.height / 2, speedyRef.width, speedyRef.height * 0.7)
    
    // Draw head
    const headColor = { r: 255, g: 235, b: 59 }
    const litHeadColor = applyPhongLighting(headColor, normal, { x: speedyRef.x, y: speedyRef.y })
    context.fillStyle = litHeadColor
    context.beginPath()
    context.arc(0, -speedyRef.height / 3, speedyRef.width / 2, 0, Math.PI * 2)
    context.fill()
    
    // Draw hat (sombrero)
    context.fillStyle = '#8B4513'
    context.fillRect(-speedyRef.width / 2 - 5, -speedyRef.height / 2 - 10, speedyRef.width + 10, 8)
    
    // Draw eyes
    context.fillStyle = 'black'
    context.beginPath()
    context.arc(-8, -speedyRef.height / 3, 3, 0, Math.PI * 2)
    context.fill()
    context.beginPath()
    context.arc(8, -speedyRef.height / 3, 3, 0, Math.PI * 2)
    context.fill()
    
    // Draw tail (animated)
    const tailWave = Math.sin(speedyRef.animationFrame) * 0.3
    context.strokeStyle = litBodyColor
    context.lineWidth = 8
    context.beginPath()
    context.moveTo(-speedyRef.width / 2, speedyRef.height / 4)
    context.quadraticCurveTo(-speedyRef.width - 10, speedyRef.height / 4 + tailWave * 10, -speedyRef.width - 20, speedyRef.height / 4 - 5)
    context.stroke()
    
    // Draw running legs (animated)
    const legOffset = Math.sin(speedyRef.animationFrame * 2) * 5
    context.strokeStyle = litBodyColor
    context.lineWidth = 6
    context.beginPath()
    context.moveTo(-10, speedyRef.height / 2)
    context.lineTo(-10 + legOffset, speedyRef.height / 2 + 15)
    context.stroke()
    context.beginPath()
    context.moveTo(10, speedyRef.height / 2)
    context.lineTo(10 - legOffset, speedyRef.height / 2 + 15)
    context.stroke()
    
    context.restore()
    
    // Draw speed lines for effect
    if (Math.sqrt(speedyRef.vx * speedyRef.vx + speedyRef.vy * speedyRef.vy) > 1) {
      context.strokeStyle = 'rgba(255, 255, 255, 0.6)'
      context.lineWidth = 2
      for (let i = 0; i < 3; i++) {
        const lineX = screenX - speedyRef.vx * (i + 1) * 5
        const lineY = screenY - speedyRef.vy * (i + 1) * 5
        context.beginPath()
        context.moveTo(lineX, lineY)
        context.lineTo(lineX - speedyRef.vx * 10, lineY - speedyRef.vy * 10)
        context.stroke()
      }
    }
  }
}

function drawUI() {
  const context = ctx.value
  const canvas = canvasRef.value
  
  // Draw minimap
  const minimapX = canvas.width - 200
  const minimapY = 10
  const minimapWidth = 180
  const minimapHeight = 135
  const scaleX = minimapWidth / VIRTUAL_WIDTH
  const scaleY = minimapHeight / VIRTUAL_HEIGHT
  
  // Minimap background
  context.fillStyle = 'rgba(0, 0, 0, 0.5)'
  context.fillRect(minimapX, minimapY, minimapWidth, minimapHeight)
  
  // Minimap obstacles
  context.fillStyle = 'rgba(139, 69, 19, 0.8)'
  for (const obstacle of obstacles.value) {
    context.fillRect(
      minimapX + obstacle.x * scaleX,
      minimapY + obstacle.y * scaleY,
      obstacle.width * scaleX,
      obstacle.height * scaleY
    )
  }
  
  // Minimap camera view
  context.strokeStyle = 'white'
  context.lineWidth = 2
  context.strokeRect(
    minimapX + camera.value.x * scaleX,
    minimapY + camera.value.y * scaleY,
    CAMERA_WIDTH * scaleX,
    CAMERA_HEIGHT * scaleY
  )
  
  // Minimap Speedy position
  context.fillStyle = 'yellow'
  context.beginPath()
  context.arc(
    minimapX + speedy.value.x * scaleX,
    minimapY + speedy.value.y * scaleY,
    3, 0, Math.PI * 2
  )
  context.fill()
  
  // Instruções
  context.fillStyle = 'white'
  context.font = '16px Arial'
  context.fillText('Controles da Câmera: WASD ou Setas do Teclado', 10, 30)
  context.fillText('Clique para capturar o Ligeirinho!', 10, 50)
  context.fillText(`Espaço Virtual: ${VIRTUAL_WIDTH}x${VIRTUAL_HEIGHT}`, 10, canvas.height - 40)
  context.fillText(`Câmera: ${Math.floor(camera.value.x)}, ${Math.floor(camera.value.y)}`, 10, canvas.height - 20)
}

// Main animation loop
function animate(currentTime) {
  const deltaTime = currentTime - lastTime
  lastTime = currentTime
  
  updateSpeedyMovement(deltaTime)
  updateCamera()
  
  // Clear and render
  const context = ctx.value
  const canvas = canvasRef.value
  context.clearRect(0, 0, canvas.width, canvas.height)
  
  drawBackground()
  drawObstacles()
  drawSpeedyGonzales()
  drawUI()
  
  animationId = requestAnimationFrame(animate)
}

// Event handlers
function handleKeyDown(event) {
  if (event.code in keys.value) {
    keys.value[event.code] = true
    event.preventDefault()
  }
}

function handleKeyUp(event) {
  if (event.code in keys.value) {
    keys.value[event.code] = false
    event.preventDefault()
  }
}

function handleMouseClick(event) {
  const canvas = canvasRef.value
  const rect = canvas.getBoundingClientRect()
  const clickX = event.clientX - rect.left + camera.value.x
  const clickY = event.clientY - rect.top + camera.value.y
  
  // Check if clicked on Speedy
  const dx = clickX - speedy.value.x
  const dy = clickY - speedy.value.y
  const distance = Math.sqrt(dx * dx + dy * dy)
  
  if (distance < 50) {
    // Caught Speedy! Make him jump away
    speedy.value.targetDirection = Math.atan2(speedy.value.y - clickY, speedy.value.x - clickX)
    speedy.value.speed = speedy.value.maxSpeed * 1.5
    
    // Reset speed after a moment
    setTimeout(() => {
      speedy.value.speed = 2
    }, 1000)
  }
}

// Lifecycle
onMounted(() => {
  const canvas = canvasRef.value
  canvas.width = CAMERA_WIDTH
  canvas.height = CAMERA_HEIGHT
  ctx.value = canvas.getContext('2d')
  
  // Add event listeners
  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', handleKeyUp)
  canvas.addEventListener('click', handleMouseClick)
  
  // Start animation
  lastTime = performance.now()
  animate(lastTime)
})

onUnmounted(() => {
  if (animationId) {
    cancelAnimationFrame(animationId)
  }
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
})
</script>

<template>
  <div class="game-container">
    <h1>Ligeirinho - Cena Gráfica 2D Interativa</h1>
    <div class="main-content">
      <canvas 
        ref="canvasRef"
        class="game-canvas"
        :width="CAMERA_WIDTH"
        :height="CAMERA_HEIGHT"
      ></canvas>
      <div class="info-panel">
        <h3>Recursos de Computação Gráfica Demonstrados:</h3>
        <ul>
          <li><strong>Espaço Virtual:</strong> Mundo {{ VIRTUAL_WIDTH }}x{{ VIRTUAL_HEIGHT }}, viewport da câmera {{ CAMERA_WIDTH }}x{{ CAMERA_HEIGHT }}</li>
          <li><strong>Iluminação Phong:</strong> Iluminação ambiente, difusa e especular em todos os objetos</li>
          <li><strong>Transformações Afins:</strong> Translação, rotação e escala para animação do personagem</li>
          <li><strong>IA Autônoma:</strong> Busca de caminho, desvio de obstáculos, movimento randomizado</li>
          <li><strong>Projeção de Câmera:</strong> Movimento independente da câmera com transições suaves</li>
          <li><strong>Recursos Interativos:</strong> Clique para interagir com o Ligeirinho, controles de câmera por teclado</li>
        </ul>
        <div class="controls-info">
          <h4>Como Jogar:</h4>
          <p><strong>WASD</strong> ou <strong>Setas:</strong> Mover câmera</p>
          <p><strong>Click:</strong> Capturar Ligeirinho</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  font-family: 'Arial', sans-serif;
}

h1 {
  color: white;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
  margin-bottom: 20px;
  text-align: center;
}

.main-content {
  display: flex;
  gap: 20px;
  align-items: flex-start;
  justify-content: center;
  flex-wrap: wrap;
}

.game-canvas {
  border: 4px solid #fff;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.3);
  cursor: crosshair;
  background: #000;
  flex-shrink: 0;
}

.game-canvas:hover {
  box-shadow: 0 15px 40px rgba(0,0,0,0.4);
}

.info-panel {
  background: rgba(255, 255, 255, 0.95);
  padding: 20px;
  border-radius: 10px;
  max-width: 350px;
  min-width: 300px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
  backdrop-filter: blur(5px);
}

.info-panel h3 {
  color: #2c3e50;
  margin-top: 0;
  margin-bottom: 15px;
  font-size: 1.1em;
  border-bottom: 2px solid #667eea;
  padding-bottom: 8px;
}

.info-panel h4 {
  color: #34495e;
  margin-top: 20px;
  margin-bottom: 10px;
  font-size: 1em;
}

.info-panel ul {
  text-align: left;
  color: #444;
  padding-left: 20px;
  margin: 0;
}

.info-panel li {
  margin-bottom: 12px;
  line-height: 1.5;
  color: #555;
}

.info-panel strong {
  color: #2980b9;
  font-weight: 600;
}

.controls-info {
  background: rgba(52, 152, 219, 0.1);
  padding: 15px;
  border-radius: 8px;
  margin-top: 15px;
  border-left: 4px solid #3498db;
}

.controls-info p {
  margin: 8px 0;
  color: #2c3e50;
  font-size: 0.9em;
}

.controls-info strong {
  color: #1e3a8a;
  background: rgba(59, 130, 246, 0.1);
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
}

/* Responsividade */
@media (max-width: 1200px) {
  .main-content {
    flex-direction: column;
    align-items: center;
  }
  
  .info-panel {
    max-width: 800px;
    width: 100%;
  }
}

@media (max-width: 850px) {
  .game-canvas {
    transform: scale(0.8);
    transform-origin: center;
  }
  
  .info-panel {
    max-width: 90%;
  }
}
</style>
