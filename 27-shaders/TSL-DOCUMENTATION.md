# Three.js Shading Language (TSL) - Guía Completa

> **Última actualización:** Enero 2025  
> **Versión Three.js:** 0.174.0+

## 📚 Tabla de Contenidos

1. [Introducción](#introducción)
2. [¿Qué es TSL?](#qué-es-tsl)
3. [GLSL vs TSL: Comparación Detallada](#glsl-vs-tsl-comparación-detallada)
4. [Instalación y Configuración](#instalación-y-configuración)
5. [Conceptos Fundamentales](#conceptos-fundamentales)
6. [Migración de GLSL a TSL](#migración-de-glsl-a-tsl)
7. [Ejemplos Prácticos](#ejemplos-prácticos)
8. [WebGPU y TSL](#webgpu-y-tsl)
9. [Recursos y Enlaces Útiles](#recursos-y-enlaces-útiles)

---

## Introducción

**Three.js Shading Language (TSL)** es un sistema moderno de shaders para Three.js que permite crear efectos visuales complejos usando una sintaxis basada en nodos y JavaScript/TypeScript. TSL está diseñado para ser compatible tanto con WebGL como con WebGPU, compilando automáticamente a GLSL o WGSL según el renderer utilizado.

### ¿Por qué TSL?

- ✅ **Sintaxis familiar**: Similar a JavaScript/TypeScript
- ✅ **Modular y reutilizable**: Sistema de nodos componible
- ✅ **Compatible con WebGPU**: Se compila automáticamente a WGSL
- ✅ **Mejor integración**: Funciona nativamente con Three.js
- ✅ **Editor visual**: Herramientas como Shader Graph (beta)

---

## ¿Qué es TSL?

TSL es un sistema de shaders que abstrae la complejidad de GLSL/WGSL mediante un sistema de nodos. En lugar de escribir código de shader directamente, construyes shaders conectando nodos que representan operaciones matemáticas, texturas, colores, etc.

### Características Principales

1. **Sistema de Nodos**: Construye shaders conectando nodos funcionales
2. **Compilación Automática**: TSL → GLSL (WebGL) o WGSL (WebGPU)
3. **TypeScript Support**: Tipado completo disponible
4. **Hot Module Replacement**: Compatible con Vite y otros bundlers
5. **Debugging Mejorado**: Mejor integración con herramientas de desarrollo

---

## GLSL vs TSL: Comparación Detallada

### Comparación General

| Aspecto | GLSL | TSL |
|---------|------|-----|
| **Sintaxis** | Similar a C | Similar a JavaScript |
| **Lenguaje** | Lenguaje separado | Integrado en JavaScript |
| **Modularidad** | Baja (código monolítico) | Alta (sistema de nodos) |
| **Reutilización** | Difícil | Fácil (nodos reutilizables) |
| **Debugging** | Difícil | Más fácil (herramientas JS) |
| **WebGPU** | No compatible | Compatible (compila a WGSL) |
| **Curva de aprendizaje** | Media-Alta | Media (si sabes JS) |
| **Control** | Total | Alto (con algunas abstracciones) |

### Comparación de Código

#### Ejemplo 1: Shader Simple con Color

**GLSL:**
```glsl
// vertex.glsl
void main() {
    gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4(position, 1.0);
}

// fragment.glsl
uniform vec3 uColor;

void main() {
    gl_FragColor = vec4(uColor, 1.0);
}
```

**TSL (equivalente):**
```javascript
import { MeshNodeMaterial, uniform, vec3, vec4, positionLocal, modelViewProjection } from 'three/nodes'

const uColor = uniform(new THREE.Color('orange'))
const material = new MeshNodeMaterial()
material.colorNode = vec4(uColor, 1.0)
material.vertexNode = modelViewProjection(positionLocal)
```

#### Ejemplo 2: Shader con Textura y Animación

**GLSL:**
```glsl
// vertex.glsl
uniform vec2 uFrequency;
uniform float uTime;
varying vec2 vUv;
varying float vElevation;

void main() {
    vec4 modelPosition = modelMatrix * vec4(position, 1.0);
    float elevation = sin(modelPosition.x * uFrequency.x - uTime) * 0.1;
    modelPosition.z += elevation;
    
    gl_Position = projectionMatrix * viewMatrix * modelPosition;
    vUv = uv;
    vElevation = elevation;
}

// fragment.glsl
uniform sampler2D uTexture;
varying vec2 vUv;
varying float vElevation;

void main() {
    vec4 textureColor = texture2D(uTexture, vUv);
    textureColor.rgb *= vElevation * 2.0 + 0.5;
    gl_FragColor = textureColor;
}
```

**TSL (equivalente):**
```javascript
import { 
    MeshNodeMaterial,
    uniform,
    texture,
    uv,
    vec2,
    vec3,
    vec4,
    sin,
    add,
    mul,
    positionLocal,
    modelViewProjection
} from 'three/nodes'

// Uniforms
const uFrequency = uniform(new THREE.Vector2(10, 5))
const uTime = uniform(0)
const uTexture = texture(flagTexture)

// Material
const material = new MeshNodeMaterial()

// Vertex Shader
const position = positionLocal
const frequency = vec2(uFrequency.x, uFrequency.y)
const time = uTime

const elevationX = sin(add(mul(position.x, frequency.x), mul(time, -1))) * 0.1
const elevationY = sin(add(mul(position.y, frequency.y), mul(time, -1))) * 0.1
const elevation = add(elevationX, elevationY)

const modifiedPosition = vec3(position.x, position.y, add(position.z, elevation))
material.vertexNode = modelViewProjection(modifiedPosition)

// Fragment Shader
const vUv = uv()
const textureColor = uTexture.sample(vUv)
const elevationFactor = add(mul(elevation, 2.0), 0.5)
const finalColor = mul(textureColor.rgb, elevationFactor)

material.colorNode = vec4(finalColor, textureColor.a)
```

### Ventajas y Desventajas

#### GLSL
**Ventajas:**
- ✅ Control total sobre la GPU
- ✅ Estándar establecido y ampliamente usado
- ✅ Muchos recursos y ejemplos disponibles
- ✅ Compatible con todos los navegadores
- ✅ Sin overhead de abstracción

**Desventajas:**
- ❌ Sintaxis tipo C (puede ser confusa)
- ❌ Lenguaje separado (no es JavaScript)
- ❌ Menos modular
- ❌ Debugging más difícil
- ❌ No compatible con WebGPU directamente

#### TSL
**Ventajas:**
- ✅ Sintaxis familiar (JavaScript/TypeScript)
- ✅ Modular y reutilizable
- ✅ Mejor integración con Three.js
- ✅ Compatible con WebGPU (compila a WGSL)
- ✅ Editor visual disponible (Shader Graph)
- ✅ Mejor debugging

**Desventajas:**
- ❌ Más nuevo (menos recursos)
- ❌ Abstracción adicional (menos control directo)
- ❌ Requiere entender el sistema de nodos
- ❌ Puede tener overhead mínimo

---

## Instalación y Configuración

### Requisitos

- **Three.js**: Versión 0.150.0 o superior (recomendado 0.174.0+)
- **Node.js**: 16.0.0 o superior
- **Bundler**: Vite, Webpack, o similar

### Instalación

TSL viene incluido en Three.js, no necesitas instalar nada adicional:

```bash
npm install three
```

### Configuración con Vite

Si usas Vite, no necesitas configuración especial. TSL funciona directamente:

```javascript
// vite.config.js
import { defineConfig } from 'vite'

export default defineConfig({
    // TSL funciona sin configuración adicional
})
```

### Importación de Nodos

```javascript
// Importación básica
import { 
    MeshNodeMaterial,
    uniform,
    texture,
    uv,
    vec2,
    vec3,
    vec4,
    sin,
    cos,
    add,
    mul,
    sub,
    div,
    positionLocal,
    modelViewProjection
} from 'three/nodes'
```

---

## Conceptos Fundamentales

### 1. Nodos (Nodes)

Los nodos son las unidades básicas de TSL. Cada nodo representa una operación o valor:

```javascript
// Nodo de valor uniforme
const uTime = uniform(0)

// Nodo de posición
const position = positionLocal

// Nodo de operación matemática
const result = add(position.x, 1.0)
```

### 2. Uniforms

Los uniforms se crean usando la función `uniform()`:

```javascript
// Uniform simple
const uTime = uniform(0)
const uColor = uniform(new THREE.Color('orange'))
const uFrequency = uniform(new THREE.Vector2(10, 5))

// Actualizar uniform
uTime.value = elapsedTime
```

### 3. Materiales con Nodos

Usa `MeshNodeMaterial` en lugar de `ShaderMaterial`:

```javascript
import { MeshNodeMaterial } from 'three/nodes'

const material = new MeshNodeMaterial()
material.colorNode = vec4(1, 0, 0, 1) // Color rojo
material.vertexNode = modelViewProjection(positionLocal)
```

### 4. Operaciones Matemáticas

```javascript
// Suma
const sum = add(a, b)

// Multiplicación
const product = mul(a, b)

// Resta
const difference = sub(a, b)

// División
const quotient = div(a, b)

// Funciones trigonométricas
const sine = sin(angle)
const cosine = cos(angle)
```

### 5. Texturas

```javascript
import { texture } from 'three/nodes'

const uTexture = texture(flagTexture)
const vUv = uv()
const textureColor = uTexture.sample(vUv)
```

### 6. Vectores

```javascript
import { vec2, vec3, vec4 } from 'three/nodes'

const v2 = vec2(1.0, 2.0)
const v3 = vec3(1.0, 2.0, 3.0)
const v4 = vec4(1.0, 2.0, 3.0, 1.0)

// Acceso a componentes
const x = v3.x
const y = v3.y
const z = v3.z
```

---

## Migración de GLSL a TSL

### Paso a Paso: Convertir tu Shader Actual

#### Paso 1: Identificar Componentes GLSL

Analiza tu shader GLSL:
- **Uniforms**: `uniform vec2 uFrequency;`
- **Varyings**: `varying vec2 vUv;`
- **Operaciones**: `sin()`, `*`, `+`, etc.
- **Texturas**: `texture2D()`
- **Matrices**: `modelMatrix`, `viewMatrix`, etc.

#### Paso 2: Convertir Uniforms

**GLSL:**
```glsl
uniform vec2 uFrequency;
uniform float uTime;
uniform vec3 uColor;
uniform sampler2D uTexture;
```

**TSL:**
```javascript
const uFrequency = uniform(new THREE.Vector2(10, 5))
const uTime = uniform(0)
const uColor = uniform(new THREE.Color('orange'))
const uTexture = texture(flagTexture)
```

#### Paso 3: Convertir Vertex Shader

**GLSL:**
```glsl
void main() {
    vec4 modelPosition = modelMatrix * vec4(position, 1.0);
    float elevation = sin(modelPosition.x * uFrequency.x - uTime) * 0.1;
    modelPosition.z += elevation;
    gl_Position = projectionMatrix * viewMatrix * modelPosition;
}
```

**TSL:**
```javascript
const position = positionLocal
const elevation = sin(add(mul(position.x, uFrequency.x), mul(uTime, -1))) * 0.1
const modifiedPosition = vec3(position.x, position.y, add(position.z, elevation))
material.vertexNode = modelViewProjection(modifiedPosition)
```

#### Paso 4: Convertir Fragment Shader

**GLSL:**
```glsl
void main() {
    vec4 textureColor = texture2D(uTexture, vUv);
    gl_FragColor = textureColor;
}
```

**TSL:**
```javascript
const vUv = uv()
const textureColor = uTexture.sample(vUv)
material.colorNode = textureColor
```

### Guía de Conversión Rápida

| GLSL | TSL |
|------|-----|
| `uniform vec3 uColor;` | `const uColor = uniform(new THREE.Color('orange'))` |
| `varying vec2 vUv;` | `const vUv = uv()` |
| `texture2D(uTexture, vUv)` | `uTexture.sample(vUv)` |
| `vec4(position, 1.0)` | `vec4(position, 1.0)` |
| `a * b` | `mul(a, b)` |
| `a + b` | `add(a, b)` |
| `a - b` | `sub(a, b)` |
| `sin(x)` | `sin(x)` |
| `cos(x)` | `cos(x)` |
| `modelMatrix * vec4(...)` | `modelViewProjection(position)` |
| `gl_Position` | `material.vertexNode` |
| `gl_FragColor` | `material.colorNode` |

---

## Ejemplos Prácticos

### Ejemplo 1: Shader Básico con Color

```javascript
import * as THREE from 'three'
import { MeshNodeMaterial, uniform, vec4, positionLocal, modelViewProjection } from 'three/nodes'

const geometry = new THREE.PlaneGeometry(1, 1)
const material = new MeshNodeMaterial()

// Vertex shader
material.vertexNode = modelViewProjection(positionLocal)

// Fragment shader
const uColor = uniform(new THREE.Color('red'))
material.colorNode = vec4(uColor, 1.0)

const mesh = new THREE.Mesh(geometry, material)
scene.add(mesh)
```

### Ejemplo 2: Shader con Textura Animada

```javascript
import { 
    MeshNodeMaterial,
    uniform,
    texture,
    uv,
    vec2,
    vec3,
    vec4,
    sin,
    add,
    mul,
    positionLocal,
    modelViewProjection
} from 'three/nodes'

const textureLoader = new THREE.TextureLoader()
const flagTexture = textureLoader.load('/textures/ukraine.png')

const geometry = new THREE.PlaneGeometry(1, 1, 32, 32)
const material = new MeshNodeMaterial()

// Uniforms
const uFrequency = uniform(new THREE.Vector2(10, 5))
const uTime = uniform(0)
const uTexture = texture(flagTexture)

// Vertex Shader
const position = positionLocal
const frequency = vec2(uFrequency.x, uFrequency.y)
const time = uTime

const elevationX = sin(add(mul(position.x, frequency.x), mul(time, -1))) * 0.1
const elevationY = sin(add(mul(position.y, frequency.y), mul(time, -1))) * 0.1
const elevation = add(elevationX, elevationY)

const modifiedPosition = vec3(position.x, position.y, add(position.z, elevation))
material.vertexNode = modelViewProjection(modifiedPosition)

// Fragment Shader
const vUv = uv()
const textureColor = uTexture.sample(vUv)
const elevationFactor = add(mul(elevation, 2.0), 0.5)
const finalColor = mul(textureColor.rgb, elevationFactor)

material.colorNode = vec4(finalColor, textureColor.a)

// Animación
const clock = new THREE.Clock()
const tick = () => {
    uTime.value = clock.getElapsedTime()
    renderer.render(scene, camera)
    requestAnimationFrame(tick)
}
tick()
```

### Ejemplo 3: Shader con Múltiples Texturas

```javascript
import { MeshNodeMaterial, texture, uv, mix, vec4, positionLocal, modelViewProjection } from 'three/nodes'

const texture1 = texture(textureLoader.load('/textures/texture1.jpg'))
const texture2 = texture(textureLoader.load('/textures/texture2.jpg'))
const blendFactor = uniform(0.5)

const material = new MeshNodeMaterial()
material.vertexNode = modelViewProjection(positionLocal)

const vUv = uv()
const color1 = texture1.sample(vUv)
const color2 = texture2.sample(vUv)
const blended = mix(color1, color2, blendFactor)

material.colorNode = blended
```

---

## WebGPU y TSL

### ¿Qué es WebGPU?

WebGPU es la evolución de WebGL, ofreciendo mejor rendimiento y más control sobre la GPU. WebGPU usa WGSL (WebGPU Shading Language) en lugar de GLSL.

### TSL y WebGPU

TSL está diseñado para ser compatible con ambos:
- **WebGL**: TSL compila a GLSL
- **WebGPU**: TSL compila a WGSL automáticamente

### Usar WebGPURenderer con TSL

```javascript
import { WebGPURenderer } from 'three/addons/renderers/WebGPURenderer.js'
import { MeshNodeMaterial } from 'three/nodes'

// Detectar WebGPU
let renderer

async function initRenderer() {
    if (WebGPURenderer && await WebGPURenderer.isAvailable()) {
        renderer = new WebGPURenderer({ canvas: canvas })
        console.log('✅ Usando WebGPU (TSL → WGSL)')
    } else {
        renderer = new THREE.WebGLRenderer({ canvas: canvas })
        console.log('⚠️ Usando WebGL (TSL → GLSL)')
    }
    
    renderer.setSize(window.innerWidth, window.innerHeight)
    return renderer
}

// Tu material TSL funcionará con ambos
const material = new MeshNodeMaterial()
// ... configuración TSL ...

// Inicializar
initRenderer().then(() => {
    // Tu aplicación funciona igual, sin importar el renderer
})
```

### Ventajas de Rendimiento

- **2.5x a 3x** más rápido en renderizado general
- **Hasta 10x** más rápido en escenas complejas
- **Hasta 13x** más partículas a 60 fps
- Mejor paralelismo y control de la GPU

---

## Recursos y Enlaces Útiles

### Documentación Oficial

- **[Three.js Documentation](https://threejs.org/docs/)** - Documentación oficial de Three.js
- **[Three.js Nodes Reference](https://github.com/mrdoob/three.js/blob/dev/src/nodes/Nodes.js)** - Lista completa de nodos disponibles
- **[Three.js Examples](https://threejs.org/examples/)** - Ejemplos oficiales (busca "webgpu" o "tsl")

### Anuncios y Introducción

- **[Official TSL Announcement](https://github.com/mrdoob/three.js/wiki/Three.js-Shading-Language)** - Anuncio oficial y introducción
- **[Q&A sobre TSL](https://github.com/boytchev/tsl-textures/wiki/Q&A)** - Preguntas frecuentes

### Tutoriales y Aprendizaje

- **[SBCODE TSL Tutorials](https://sbcode.net/tsl/)** - Tutoriales detallados de Sean Bradley
- **[Udemy Course: TSL and Signed Distance Fields](https://www.udemy.com/course/threejs-shading-language/)** - Curso completo en Udemy
- **[YouTube: TSL Overview](https://www.youtube.com/watch?v=73quCt_NQMA)** - Video introductorio

### Ejemplos y Demos

- **[Three.js WebGPU Examples](https://threejs.org/examples/?q=webgpu#webgpu_parallax_uv)** - Ejemplos oficiales con WebGPU
- **[TSL Textures Repository](https://github.com/boytchev/tsl-textures)** - Texturas creadas con TSL
- **[TSL Textures Demos](https://boytchev.github.io/tsl-textures/)** - Demos interactivos
- **[Three.js TSL Tutorials (GitHub)](https://github.com/cmhhelgeson/Threejs_TSL_Tutorials)** - Repositorio con ejemplos

### Herramientas de Conversión

- **[TSL Editor (TSL → WGSL/GLSL)](https://threejs.org/examples/?q=webgpu#webgpu_tsl_editor)** - Editor online de TSL
- **[GLSL → TSL Transpiler](https://threejs.org/examples/?q=webgpu#webgpu_tsl_transpiler)** - Convierte GLSL a TSL

### Editores Visuales

- **[Three.js Shader Graph](https://www.threejsshadergraph.com/)** - Editor visual de nodos (Beta)
- **[NodeToy](https://nodetoy.co/)** - Editor visual alternativo

### Comunidad

- **[Three.js Discourse](https://discourse.threejs.org/)** - Foro oficial de Three.js
- **[Three.js GitHub](https://github.com/mrdoob/three.js)** - Repositorio oficial

### WebGPU Resources

- **[WebGPU Specification](https://www.w3.org/TR/webgpu/)** - Especificación oficial
- **[WebGPU Samples](https://webgpu.github.io/webgpu-samples/)** - Ejemplos de WebGPU
- **[Learn WebGPU](https://eliemichel.github.io/LearnWebGPU/)** - Tutorial completo de WebGPU

---

## Notas Finales

### Estado Actual (Enero 2025)

- ✅ TSL está estable y listo para producción
- ✅ Compatible con Three.js 0.150.0+
- ✅ WebGPU soportado en Chrome, Edge, Firefox, Safari
- ⚠️ Shader Graph aún en beta
- 📈 Desarrollo activo, actualizaciones frecuentes

### Recomendaciones

1. **Aprende GLSL primero**: Te dará fundamentos sólidos
2. **Luego explora TSL**: Será más fácil entender los conceptos
3. **Experimenta con WebGPU**: Cuando domines TSL, prueba WebGPURenderer
4. **Mantente actualizado**: TSL evoluciona rápidamente

### Próximos Pasos

1. Completa tu curso actual con GLSL
2. Revisa los tutoriales de SBCODE sobre TSL
3. Convierte uno de tus shaders GLSL a TSL
4. Experimenta con WebGPURenderer
5. Únete a la comunidad en Discourse

---

**¡Feliz coding con TSL! 🚀**

