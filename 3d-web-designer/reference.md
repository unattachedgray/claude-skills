# 3D Web Designer — Complete Reference

## Table of Contents
1. [Drei Component Catalog](#drei-component-catalog)
2. [Canvas & Hooks API](#canvas--hooks-api)
3. [Model Loading Pipeline](#model-loading-pipeline)
4. [Shader Patterns](#shader-patterns)
5. [Camera Controls](#camera-controls)
6. [Scroll-Driven Scenes](#scroll-driven-scenes)
7. [Post-Processing Effects](#post-processing-effects)
8. [Physics](#physics)
9. [Animation Techniques](#animation-techniques)
10. [HTML Overlays in 3D](#html-overlays-in-3d)
11. [Typography in 3D](#typography-in-3d)
12. [Material Recipes](#material-recipes)
13. [Performance Optimization](#performance-optimization)
14. [Design Tools & Pipeline](#design-tools--pipeline)
15. [Accessibility](#accessibility)

---

## Drei Component Catalog

### Cameras
PerspectiveCamera, OrthographicCamera, CubeCamera

### Controls
CameraControls, ScrollControls, PresentationControls, KeyboardControls, FaceControls, MotionPathControls, OrbitControls, MapControls, FlyControls, PointerLockControls

### Gizmos
GizmoHelper, PivotControls, DragControls, TransformControls, Grid, Helper/useHelper

### Shapes
Plane, Box, Sphere, Circle, Cone, Cylinder, Tube, Torus, TorusKnot, Ring, Tetrahedron, Polyhedron, Icosahedron, Octahedron, Dodecahedron, Extrude, Lathe, Shape, RoundedBox, ScreenQuad, Line, QuadraticBezierLine, CubicBezierLine, CatmullRomLine, Facemesh

### Abstractions
Image, Text, Text3D, Effects, PositionalAudio, Billboard, ScreenSpace, ScreenSizer, GradientTexture, Edges, Outlines, Trail, Sampler, ComputedAttribute, Clone, useAnimations, MarchingCubes, Decal, Svg, AsciiRenderer, Splat

### Shaders / Materials
MeshReflectorMaterial, MeshWobbleMaterial, MeshDistortMaterial, MeshRefractionMaterial, MeshTransmissionMaterial, MeshDiscardMaterial, PointMaterial, SoftShadows, shaderMaterial

### Loading
Loader, Progress/useProgress, Gltf/useGLTF, Fbx/useFBX, Texture/useTexture, Ktx2/useKTX2, CubeTexture/useCubeTexture, VideoTexture/useVideoTexture, TrailTexture/useTrailTexture, useFont, useSpriteLoader

### Performance
Instances, Merged, Points, Segments, Detailed (LOD), Preload, BakeShadows, meshBounds, AdaptiveDpr, AdaptiveEvents, Bvh, PerformanceMonitor

### Portals
Hud, View, RenderTexture, RenderCubeTexture, Fisheye, Mask, MeshPortalMaterial

### Staging
Center, Resize, BBAnchor, Bounds, CameraShake, Float, Stage, Backdrop, Shadow, Caustics, ContactShadows, RandomizedLight, AccumulativeShadows, SpotLight, SpotLightShadow, Environment, Lightformer, Sky, Stars, Sparkles, Cloud, useEnvironment, MatcapTexture/useMatcapTexture, NormalTexture/useNormalTexture, ShadowAlpha

### Misc
useContextBridge, Html, CycleRaycast, Select, Sprite Animator, Stats, StatsGl, Wireframe, useDepthBuffer, Fbo/useFBO, useCamera, DetectGPU/useDetectGPU, useAspect, useCursor, useIntersect, useBoxProjectedEnv, useSurfaceSampler, FaceLandmarker

---

## Canvas & Hooks API

### Canvas Props

```tsx
<Canvas
  camera={{ position: [0, 0, 5], fov: 75 }}
  shadows                          // enable shadow maps
  dpr={[1, 2]}                     // device pixel ratio range
  gl={{ antialias: true }}         // WebGLRenderer options
  frameloop="always"               // "always" | "demand" | "never"
  orthographic                     // use OrthographicCamera
  flat                             // disable tone mapping
  linear                           // disable color management
  // WebGPU:
  gl={async (props) => {
    const r = new THREE.WebGPURenderer(props);
    await r.init();
    return r;
  }}
>
```

### Hooks

| Hook | Returns | Usage |
|------|---------|-------|
| `useFrame((state, delta) => {})` | void | Render loop. Mutate refs, never setState. |
| `useThree()` | `{ gl, scene, camera, size, viewport, clock, ... }` | Reactive access to root state |
| `useLoader(Loader, url)` | loaded data | Suspense-based asset loading |
| `useGraph(object)` | `{ nodes, materials }` | Flatten object graph by name |

**useFrame state object:**
- `state.clock` — THREE.Clock instance
- `state.camera` — active camera
- `state.mouse` — normalized mouse position (-1 to 1)
- `state.viewport` — { width, height, factor } in world units
- `state.size` — { width, height } in pixels
- `state.gl` — renderer
- `state.scene` — root scene
- `state.invalidate()` — request render (for `frameloop="demand"`)

---

## Model Loading Pipeline

### Blender to Web

1. Model in Blender (web-friendly poly counts: 10k-100k for hero assets)
2. Bake high-poly detail to normal maps
3. Export as `.glb` with Draco compression enabled
4. Post-process: `npx gltfjsx model.glb --transform --types`
5. Load in R3F with `useGLTF`

### Compression Methods

| Method | Target | Reduction |
|--------|--------|-----------|
| Draco | Mesh geometry | 90-95% |
| Meshopt | Geometry + morph + animation | 70-90% |
| KTX2/Basis | Textures (GPU-compressed) | ~10x VRAM |
| gltfjsx --transform | Full pipeline | 70-90% total |

### Draco Setup

```tsx
import { useGLTF } from '@react-three/drei';
useGLTF.setDecoderPath('/draco/'); // host decoder files locally
```

### Progressive Loading

```tsx
<Suspense fallback={<Loader />}>
  <Suspense fallback={<LowResModel />}>
    <HighResModel />
  </Suspense>
</Suspense>
```

### Tools
- [gltf.pmnd.rs](https://gltf.pmnd.rs/) — Web tool for GLTF to JSX
- [gltf-transform.dev](https://gltf-transform.dev/) — CLI for GLT post-processing
- [optimizeglb.com](https://optimizeglb.com/) — Online GLB optimizer
- [polyhaven.com](https://polyhaven.com/) — Free HDRI, textures, models

---

## Shader Patterns

### A. Drei shaderMaterial (GLSL)

```tsx
import { shaderMaterial } from '@react-three/drei';
import { extend, useFrame } from '@react-three/fiber';

const WaveMaterial = shaderMaterial(
  { uTime: 0, uColor: new THREE.Color(0.2, 0.0, 0.1) },
  // Vertex
  /*glsl*/ `
    varying vec2 vUv;
    uniform float uTime;
    void main() {
      vUv = uv;
      vec3 pos = position;
      pos.z += sin(pos.x * 4.0 + uTime) * 0.1;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
    }
  `,
  // Fragment
  /*glsl*/ `
    uniform float uTime;
    uniform vec3 uColor;
    varying vec2 vUv;
    void main() {
      gl_FragColor = vec4(0.5 + 0.3 * sin(vUv.yxx + uTime) + uColor, 1.0);
    }
  `
);
extend({ WaveMaterial });

function Wave() {
  const ref = useRef();
  useFrame((_, delta) => (ref.current.uTime += delta));
  return (
    <mesh>
      <planeGeometry args={[4, 4, 32, 32]} />
      <waveMaterial ref={ref} key={WaveMaterial.key} uColor="hotpink" />
    </mesh>
  );
}
```

**`key={WaveMaterial.key}`** enables shader hot-reload in dev.

### B. TSL (Three Shading Language) — WebGPU-Ready

```js
import { Fn, uniform, uv, vec3, vec4, sin, positionLocal } from 'three/tsl';

const material = new THREE.MeshStandardNodeMaterial();
material.colorNode = vec4(uv(), 0.5, 1.0);
material.positionNode = positionLocal.add(
  vec3(0, sin(positionLocal.x.mul(4).add(uniform(0))).mul(0.1), 0)
);
```

TSL compiles to WGSL (WebGPU) and GLSL (WebGL) automatically.

### C. three-custom-shader-material (Extend Standard Materials)

```tsx
import CustomShaderMaterial from 'three-custom-shader-material/vanilla';
const mat = new CustomShaderMaterial({
  baseMaterial: THREE.MeshStandardMaterial,
  vertexShader: `...`,
  fragmentShader: `...`,
  uniforms: { uTime: { value: 0 } },
  // Retains PBR lighting, shadows
});
```

---

## Camera Controls

### OrbitControls
```tsx
<OrbitControls
  makeDefault
  enableDamping
  minPolarAngle={0}
  maxPolarAngle={Math.PI / 2}
  minDistance={2}
  maxDistance={20}
/>
```

### CameraControls (Animated Transitions)
```tsx
const ref = useRef();
const flyTo = () => ref.current?.setLookAt(0, 2, 5, 0, 0, 0, true);
<CameraControls ref={ref} makeDefault />
```

### PresentationControls (Drag to Rotate Object)
```tsx
<PresentationControls
  global
  cursor
  speed={1}
  zoom={0.8}
  polar={[-Math.PI / 4, Math.PI / 4]}
  azimuth={[-Math.PI / 4, Math.PI / 4]}
>
  <Model />
</PresentationControls>
```

---

## Scroll-Driven Scenes

### R3F Native (ScrollControls)

```tsx
import { ScrollControls, Scroll, useScroll } from '@react-three/drei';

function AnimatedScene() {
  const ref = useRef();
  const data = useScroll();
  useFrame(() => {
    ref.current.rotation.y = data.range(0, 1/3) * Math.PI * 2;
    ref.current.position.y = data.curve(1/3, 1/3) * 2;
  });
  return <mesh ref={ref}><boxGeometry /><meshStandardMaterial /></mesh>;
}

<Canvas>
  <ScrollControls pages={3} damping={0.1}>
    <Scroll><AnimatedScene /></Scroll>
    <Scroll html>
      <h1 style={{ position: 'absolute', top: '10vh' }}>Section 1</h1>
      <h1 style={{ position: 'absolute', top: '110vh' }}>Section 2</h1>
    </Scroll>
  </ScrollControls>
</Canvas>
```

**useScroll helpers:**
- `data.offset` — 0-1 dampened scroll position
- `data.range(from, distance)` — 0-1 within a scroll section
- `data.curve(from, distance)` — 0-1-0 bell curve
- `data.visible(from, distance)` — boolean

### GSAP ScrollTrigger (Apple-Style)

```tsx
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);

useLayoutEffect(() => {
  gsap.to(meshRef.current.rotation, {
    y: Math.PI * 2,
    scrollTrigger: { trigger: sectionRef.current, scrub: 2, pin: true },
  });
}, []);
```

---

## Post-Processing Effects

### Available Effects

| Effect | Key Props | Notes |
|--------|-----------|-------|
| Bloom | luminanceThreshold, intensity, mipmapBlur | Selective: needs `toneMapped={false}` |
| DepthOfField | focusDistance, focalLength, bokehScale | Cinematic focus |
| Noise | opacity | Film grain |
| Vignette | offset, darkness | Edge darkening |
| ChromaticAberration | offset | Color fringing |
| SSAO | radius, intensity | Ambient occlusion |
| GodRays | sun (mesh ref) | Volumetric light |
| Glitch | delay, duration, strength | Digital distortion |

### Making Objects Glow

```tsx
// This material WILL bloom (emissive + toneMapped={false})
<meshStandardMaterial
  emissive="red"
  emissiveIntensity={2}
  toneMapped={false}
/>

// This material will NOT bloom (default tone mapping clamps to 0-1)
<meshStandardMaterial color="red" />
```

---

## Physics

### @react-three/rapier (Recommended)

```tsx
import { Physics, RigidBody, CuboidCollider } from '@react-three/rapier';

<Canvas>
  <Physics gravity={[0, -9.81, 0]}>
    <RigidBody type="dynamic" colliders="hull">
      <mesh><sphereGeometry args={[0.5]} /><meshStandardMaterial /></mesh>
    </RigidBody>
    <RigidBody type="fixed">
      <CuboidCollider args={[10, 0.1, 10]} />
    </RigidBody>
  </Physics>
</Canvas>
```

**RigidBody types:** `dynamic`, `fixed`, `kinematicPosition`, `kinematicVelocity`
**Collider types:** `cuboid`, `ball`, `trimesh`, `hull`, `heightfield`, or `false` for manual
**Events:** `onCollisionEnter`, `onCollisionExit`, `onIntersectionEnter`, `onContactForce`

---

## Animation Techniques

### @react-spring/three

```tsx
import { useSpring, animated } from '@react-spring/three';

const springs = useSpring({
  scale: active ? 1.5 : 1,
  color: active ? '#569AFF' : '#ff6d6d',
  config: { mass: 1, tension: 170, friction: 26 },
});

<animated.mesh scale={springs.scale}>
  <boxGeometry />
  <animated.meshStandardMaterial color={springs.color} />
</animated.mesh>
```

**Imperative (no re-renders):**
```tsx
const [springs, api] = useSpring(() => ({ scale: 1 }));
<animated.mesh
  scale={springs.scale}
  onPointerEnter={() => api.start({ scale: 1.5 })}
  onPointerLeave={() => api.start({ scale: 1 })}
/>
```

### GSAP (Timeline-Based)

```tsx
useLayoutEffect(() => {
  const tl = gsap.timeline({ repeat: -1 });
  tl.to(meshRef.current.position, { y: 2, duration: 1, ease: 'power2.inOut' })
    .to(meshRef.current.rotation, { y: Math.PI, duration: 1 }, '<');
  return () => tl.kill();
}, []);
```

### GLTF Embedded Animations

```tsx
import { useGLTF, useAnimations } from '@react-three/drei';

function Character() {
  const { scene, animations } = useGLTF('/character.glb');
  const { actions } = useAnimations(animations, scene);
  useEffect(() => { actions['walk']?.play(); }, [actions]);
  return <primitive object={scene} />;
}
```

---

## HTML Overlays in 3D

```tsx
import { Html } from '@react-three/drei';

// Basic overlay
<Html position={[0, 2, 0]} center>
  <div className="bg-black/80 text-white px-3 py-1 rounded text-sm">Label</div>
</Html>

// With occlusion (hides behind 3D objects)
<Html occlude onOcclude={setHidden}
  style={{ opacity: hidden ? 0 : 1, transition: 'opacity 0.5s' }}>
  <div>I hide behind objects!</div>
</Html>

// As 3D plane
<Html transform scale={0.5}><div>Rendered in 3D space</div></Html>
```

**Key props:** `center`, `fullscreen`, `distanceFactor`, `transform`, `sprite`, `occlude`, `portal`

---

## Typography in 3D

### Text3D (Extruded Geometry)

```tsx
import { Text3D, Center } from '@react-three/drei';

<Center>
  <Text3D font="/fonts/helvetiker_regular.typeface.json" size={1.5} height={0.2}
    bevelEnabled bevelThickness={0.02} bevelSize={0.02}>
    Hello World
    <meshStandardMaterial color="gold" metalness={0.8} roughness={0.2} />
  </Text3D>
</Center>
```

Requires JSON fonts from [facetype.js](http://gero3.github.io/facetype.js/).

### Text (SDF — Better for UI/Labels)

```tsx
import { Text } from '@react-three/drei';

<Text position={[0, 2, 0]} fontSize={0.5} color="white" font="/fonts/Inter-Bold.woff"
  anchorX="center" anchorY="middle" maxWidth={5}>
  Hello World
</Text>
```

Accepts `.woff`, `.ttf`, `.otf` directly. Uses Troika SDF rendering in a web worker.

---

## Material Recipes

### Glass (Transmission)

```tsx
<MeshTransmissionMaterial
  samples={16}
  resolution={256}
  transmission={1}
  roughness={0.1}
  thickness={0.5}
  ior={1.5}
  chromaticAberration={0.06}
  anisotropy={0.1}
  distortion={0.0}
  distortionScale={0.3}
  temporalDistortion={0.5}
  color="#ffffff"
/>
```

### Polished Metal

```tsx
<meshPhysicalMaterial
  metalness={1.0}
  roughness={0.15}
  envMapIntensity={1.0}
  clearcoat={1.0}
  clearcoatRoughness={0.1}
/>
// Requires <Environment> for reflections
```

### Frosted Glass

```tsx
<meshPhysicalMaterial
  transmission={1.0}
  roughness={0.4}
  thickness={1.0}
  ior={1.5}
  color="#ffffff"
/>
```

### Emissive (Neon/Glow)

```tsx
<meshStandardMaterial
  color="#000000"
  emissive="#ff0066"
  emissiveIntensity={3}
  toneMapped={false}
/>
// Combine with <Bloom> for glow effect
```

### Holographic

```tsx
<MeshDistortMaterial
  color="#88ccff"
  emissive="#4488ff"
  emissiveIntensity={0.5}
  metalness={0.8}
  roughness={0.2}
  distort={0.3}
  speed={2}
  toneMapped={false}
/>
```

---

## Performance Optimization

### Draw Call Reduction

| Technique | When | Draw Calls |
|-----------|------|------------|
| InstancedMesh | 1000+ identical objects | 1 per geometry |
| BatchedMesh | Different geometries, same material | 1 per material |
| mergeGeometries | Static geometry at load | 1 total |
| Material sharing | Identical materials | Enables GPU batching |

### Instancing in R3F

```tsx
import { Instances, Instance } from '@react-three/drei';

<Instances limit={1000}>
  <boxGeometry />
  <meshStandardMaterial />
  {positions.map((pos, i) => <Instance key={i} position={pos} />)}
</Instances>
```

### Level of Detail

```tsx
import { Detailed } from '@react-three/drei';

<Detailed distances={[0, 50, 100]}>
  <HighPoly />  {/* near */}
  <MedPoly />   {/* mid */}
  <LowPoly />   {/* far */}
</Detailed>
```

### Adaptive Quality

```tsx
import { PerformanceMonitor } from '@react-three/drei';

const [dpr, setDpr] = useState(1.5);
<Canvas dpr={dpr}>
  <PerformanceMonitor onIncline={() => setDpr(2)} onDecline={() => setDpr(1)}>
    {/* scene */}
  </PerformanceMonitor>
</Canvas>
```

### On-Demand Rendering

```tsx
<Canvas frameloop="demand">
  {/* Only renders when invalidate() is called */}
  {/* Controls and animations auto-invalidate */}
</Canvas>
```

### Texture Tips
- Use KTX2 for ~10x less GPU memory
- Always power-of-two dimensions
- Call `.dispose()` when removing textures
- Combine textures into atlases

### Lighting Budget
- Cap at 3 real-time lights
- Bake static lighting into lightmaps
- Use Environment maps for ambient (essentially free)
- PointLight shadows = 6 render passes each

---

## Design Tools & Pipeline

### Spline (spline.design)
- Browser-based 3D design with real-time collaboration
- AI text-to-3D generation (Professional tier)
- Export: React components, Three.js, GLB/GLTF
- R3F bridge: `@splinetool/r3f-spline`

### Leva (Debug GUI)
```tsx
import { useControls } from 'leva';
const { color, speed } = useControls({ color: '#ff0000', speed: { value: 1, min: 0, max: 5 } });
```

### Theatre.js (Timeline Animation)
- Professional keyframe animation editor in browser
- `@theatre/r3f` integrates directly with R3F scenes
- Export animations for production playback

### gltfjsx
```bash
npx gltfjsx model.glb --transform --types
# --transform: Draco + texture compression + dedup (70-90% smaller)
# --types: TypeScript definitions
```

---

## Accessibility

### @react-three/a11y

```tsx
import { A11y } from '@react-three/a11y';

<A11y role="button" description="Rotate globe" actionCall={rotate}>
  <Globe />
</A11y>
```

**Roles:** `content` (informational), `button` (clickable), `togglebutton` (two-state)

### Best Practices
1. Provide text alternatives for meaningful 3D content
2. Ensure keyboard navigability for interactive elements
3. Respect `prefers-reduced-motion` — disable animations
4. Maintain color contrast in UI overlays
5. Provide skip links to bypass 3D content
6. Test with screen readers (NVDA, VoiceOver)

---

## 2026 Design Patterns Cheat Sheet

### Apple-Style Product Showcase
```
GSAP ScrollTrigger (pin + scrub: 2)
  + MeshPhysicalMaterial (metalness: 1, roughness: 0.15)
  + Environment (HDRI, studio preset)
  + Lightformer (key + fill + rim)
  + ContactShadows (ground plane)
  + Camera keyframes at scroll waypoints
```

### Immersive Landing Page
```
ScrollControls (pages: 5, damping: 0.1)
  + Float (gentle bob on hero object)
  + Environment (sunset/forest)
  + Bloom (selective, emissive accents)
  + Text3D (hero heading)
  + Html (overlaid copy, occlude: true)
  + PerformanceMonitor (adaptive DPR)
```

### Glass Card UI
```
MeshTransmissionMaterial (transmission: 1, roughness: 0.1)
  + RoundedBox geometry
  + Environment (soft gradient)
  + ContactShadows
  + Html (content inside glass card)
  + Float (subtle movement)
```

### Data Visualization Globe
```
react-globe.gl or custom sphere
  + dynamic import (ssr: false)
  + GeoJSON polygons
  + scoreToColor gradient mapping
  + ArcsData for connections
  + OrbitControls (constrained)
  + Atmosphere shader
```
