<template>
  <div class="app">
    <!-- Línea de límite superior SOLO VISUAL -->
    <div class="limit-line"></div>
    <div
      v-for="shape in shapes"
      :key="shape.id"
      :class="['shape', shape.type]"
      :style="{ transform: `translate(${shape.x}px, ${shape.y}px) rotate(${shape.angle}deg)` }"
      @mousedown="handleMouseDown(shape.id)"
      @mouseup="handleMouseUp"
    ></div>
    <!-- Mostrar fuerza en la esquina -->
    <div class="force-display">
      Fuerza: {{ Math.round(force) }}
    </div>
    <!-- Barra de fuerza -->
    <div class="force-bar">
      <div :style="{ height: `${Math.min(force, MAX_FORCE) / MAX_FORCE * 100}%` }"></div>
    </div>
  </div>
</template>

<script>
const GRAVITY = 0.8;
const INITIAL_FORCE = 0;
const SHAPE_SIZE = 100;
const SCREEN_PADDING = 10;
const CEILING_THRESHOLD = 200;
const FORCE_INCREMENT_PER_SECOND = 10;
const MAX_FORCE = 100;

const getRandomPosition = () => ({
  x: Math.random() * (window.innerWidth - SHAPE_SIZE - 2 * SCREEN_PADDING) + SCREEN_PADDING,
  y: Math.random() * (window.innerHeight / 2 - SHAPE_SIZE - SCREEN_PADDING) + SCREEN_PADDING,
});

const INITIAL_SHAPES = [
  { id: 1, ...getRandomPosition(), velocityY: 0, angle: 0, force: 0, type: 'square' },
  { id: 2, ...getRandomPosition(), velocityY: 0, angle: 0, force: 0, type: 'circle' },
  { id: 3, ...getRandomPosition(), velocityY: 0, angle: 0, force: 0, type: 'triangle' }
];

export default {
  name: 'App',
  data() {
    return {
      shapes: INITIAL_SHAPES,
      pressedShapeId: null,
      holdStart: null,
      force: INITIAL_FORCE,
    };
  },
  mounted() {
    this.updateShapes();
  },
  methods: {
    updateShapes() {
      setInterval(() => {
        this.shapes = this.shapes.map(shape => {
          let newY = shape.y + shape.velocityY;
          let newVelocityY = shape.velocityY + GRAVITY;
          let newAngle = shape.angle + 5;
          let newForce = shape.force;

          if (newForce < 0) {
            newY += newForce;
            newForce += 0.5;
            if (newY <= SCREEN_PADDING + CEILING_THRESHOLD) {
              newY = SCREEN_PADDING + CEILING_THRESHOLD;
              newForce = 0;
            }
          }

          // Limitar hacia abajo (sin rebote)
          if (newY >= window.innerHeight - SHAPE_SIZE - SCREEN_PADDING) {
            newY = window.innerHeight - SHAPE_SIZE - SCREEN_PADDING;
            newVelocityY = 0; // Detener la caída al tocar el suelo
            let angle_needed = 360 - shape.angle;
            newAngle = shape.angle + angle_needed;
          }

          return { ...shape, y: newY, velocityY: newVelocityY, angle: newAngle, force: newForce };
        });
      }, 16);
    },
    updateForce() {
  if (this.forceInterval) return; // Evita múltiples intervalos
  this.forceInterval = setInterval(() => {
    if (this.pressedShapeId !== null && this.holdStart !== null) {
      const holdDuration = (Date.now() - this.holdStart) / 200;
      let newForce = FORCE_INCREMENT_PER_SECOND * holdDuration;
      newForce = Math.min(newForce, MAX_FORCE);
      this.force = newForce;
    }
  }, 30);
},

    handleMouseDown(id) {
      this.pressedShapeId = id;
      this.holdStart = Date.now();
      this.updateForce();
    },
    handleMouseUp() {
      if (this.pressedShapeId !== null) {
        const finalForce = this.force;
        console.log("fuerza + ", finalForce);

        this.shapes = this.shapes.map(shape =>
          shape.id === this.pressedShapeId ? { ...shape, force: -finalForce } : shape,
        );

        this.pressedShapeId = null;
        this.holdStart = null;
      }
    }
  }
};
</script>

<style scoped>
.app {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
.limit-line {
  position: absolute;
  top: 0;
  width: 100%;
  height: 2px;
  background-color: black;
}
.shape {
  position: absolute;
  width: 100px;
  height: 100px;
}
.shape.square {
  background-color: red;
}
.shape.circle {
  background-color: blue;
  border-radius: 50%;
}
.shape.triangle {
  background-color: green;
  clip-path: polygon(50% 0%, 100% 100%, 0% 100%);
}
.force-display {
  position: absolute;
  top: 20px;
  left: 70px;
  font-size: 20px;
  font-weight: bold;
  color: black;
}
.force-bar {
  position: absolute;
  top: 15px;
  left: 40px;
  width: 20px;
  height: 200px;
  background-color: #ccc;
  border-radius: 5px;
  overflow: hidden;
  transform: rotate(180deg);
}
.force-bar div {
  background-color: #007bff;
  transition: height;
}
</style>
