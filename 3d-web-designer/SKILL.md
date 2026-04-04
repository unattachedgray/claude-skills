---
name: 3d-web-designer
description: Use when building 3D web experiences with Three.js, React Three Fiber, or WebGPU. Use when adding 3D elements to Next.js apps, creating product showcases, scroll-driven 3D scenes, interactive globes, or immersive landing pages. Use when working with GLTF models, custom shaders, post-processing effects, or 3D animations.
---

# 3D Web Designer

## Overview

Build production-grade 3D web experiences with Three.js, React Three Fiber (R3F), and the pmndrs ecosystem. This skill covers the full pipeline: scene setup, model loading, materials, lighting, animation, post-processing, scroll integration, performance optimization, and modern design patterns.

**Core stack (2026):** Three.js r182 + @react-three/fiber v9 + @react-three/drei + Next.js (App Router)

## When to Use

- Adding 3D elements to a Next.js or React app
- Building product showcases, landing pages, or data visualizations with 3D
- Loading and displaying GLTF/GLB models
- Creating scroll-driven 3D animations
- Writing custom shaders (GLSL or TSL)
- Optimizing 3D scene performance
- Implementing glass, metal, or PBR materials
- Adding post-processing effects (bloom, DoF, etc.)

## Quick Reference

| Package | Purpose | Version |
|---------|---------|---------|
| `three` | 3D engine (WebGL + WebGPU) | r182 (0.182.0) |
| `@react-three/fiber` | React renderer for Three.js | 9.x (React 19) |
| `@react-three/drei` | 100+ helper components | 10.x |
| `@react-three/postprocessing` | Effects (bloom, DoF, etc.) | latest |
| `@react-three/rapier` | WASM physics (Rapier) | 2.x |
| `@react-spring/three` | Spring-based 3D animations | latest |
| `gltfjsx` | GLTF to React components | latest |
| `leva` | GUI debug controls | latest |
| `theatre` | Timeline animation editor | 0.5 |

## Next.js Setup (Critical)

Three.js crashes on the server. **Always use dynamic import with SSR disabled:**

```tsx
// app/page.tsx (Server Component)
import dynamic from 'next/dynamic';
const Scene = dynamic(() => import('@/components/scene'), { ssr: false });
export default function Page() { return <Scene />; }
```

```tsx
// components/scene.tsx (Client Component)
'use client';
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Environment } from '@react-three/drei';

export default function Scene() {
  return (
    <Canvas camera={{ position: [0, 0, 5] }} shadows>
      <ambientLight intensity={Math.PI / 2} />
      <mesh><boxGeometry /><meshStandardMaterial color="hotpink" /></mesh>
      <OrbitControls />
      <Environment preset="city" />
    </Canvas>
  );
}
```

**R3F v9 breaking changes:** `MeshProps` is now `ThreeElements['mesh']`. Light intensity must be multiplied by `Math.PI` (since Three.js r155).

## Core Patterns

### Loading Models

```tsx
import { useGLTF, Html, useProgress } from '@react-three/drei';

function Model() {
  const { scene } = useGLTF('/model.glb');
  return <primitive object={scene} />;
}
useGLTF.preload('/model.glb');

// Wrap in Suspense with loading indicator
<Suspense fallback={<Html center><Loader /></Html>}>
  <Model />
</Suspense>
```

Use `npx gltfjsx model.glb --transform --types` to convert GLB to typed JSX components with 70-90% size reduction (Draco + texture compression).

### Performance Rules

1. **Never `setState` in `useFrame`** -- mutate refs directly
2. **Never create objects in `useFrame`** -- hoist `new THREE.Vector3()` outside
3. **Use delta for frame-rate independence:** `ref.current.rotation.y += delta`
4. **Use InstancedMesh** for 1000+ identical objects (1 draw call)
5. **Toggle `visible`** instead of mount/unmount (avoids shader recompilation)
6. **Cap DPR at 2** on mobile; use `<PerformanceMonitor>` for adaptive quality
7. **Target < 100 draw calls** for 60fps; check `renderer.info`

### Interactions

```tsx
<mesh
  onClick={(e) => { e.stopPropagation(); /* handle */ }}
  onPointerOver={() => document.body.style.cursor = 'pointer'}
  onPointerOut={() => document.body.style.cursor = 'auto'}
>
```

Always call `e.stopPropagation()` to prevent events firing on objects behind.

### Animation

```tsx
// Spring-based (declarative, interruptible)
import { useSpring, animated } from '@react-spring/three';
const springs = useSpring({ scale: active ? 1.5 : 1 });
<animated.mesh scale={springs.scale} />

// Frame loop (simple continuous)
useFrame((state, delta) => { ref.current.rotation.y += delta; });

// GLTF embedded animations
const { actions } = useAnimations(animations, scene);
useEffect(() => { actions['walk']?.play(); }, [actions]);
```

### Post-Processing

```tsx
import { EffectComposer, Bloom, Vignette } from '@react-three/postprocessing';

<EffectComposer>
  <Bloom luminanceThreshold={0.9} intensity={1.0} />
  <Vignette offset={0.1} darkness={1.1} />
</EffectComposer>
```

**Selective bloom:** Set `emissiveIntensity > 1` and `toneMapped={false}` on materials that should glow.

## 2026 Design Trends

### Scroll-Driven 3D
- GSAP ScrollTrigger with `pin: true, scrub: 2` for Apple-style product reveals
- Drei `<ScrollControls pages={3}>` for R3F-native scroll scenes
- `useScroll().range(from, distance)` for section-based progress (0-1)

### Materials
- **Glass:** `<MeshTransmissionMaterial>` (transmission + chromatic aberration + roughness blur)
- **Metal:** `MeshPhysicalMaterial` with `metalness: 1, roughness: 0.1-0.3` + HDRI environment
- **Soft shadows:** `<ContactShadows>` for ground plane, `<AccumulativeShadows>` for baked quality

### Lighting
- Use `<Environment preset="sunset" />` (HDRI) instead of manual lights for PBR
- `<Lightformer>` for custom area lights without performance cost
- Cap real-time lights at 3; bake static lighting

### Micro-Interactions
- Hover: subtle tilt/scale/emissive shift on 3D objects
- Cursor tracking: objects follow or react to mouse position
- Click: brief scale pulse (1.0 -> 0.95 -> 1.0)

### Dark Mode
- Deep gray backgrounds (#1a1a2e) not pure black (preserves object edges)
- Lower ambient light; let emissive accents define the scene
- Glassmorphic UI overlays on dark 3D backgrounds

### Mobile-First
- Detect `navigator.deviceMemory` / `hardwareConcurrency` for quality tiers
- Replace hover interactions with tap on touch devices
- Minimum 44px touch targets
- Lazy-load 3D assets below the fold

## WebGPU (Production Ready)

```tsx
// Zero-config WebGPU with automatic WebGL 2 fallback
import * as THREE from 'three/webgpu';

<Canvas gl={async (props) => {
  const renderer = new THREE.WebGPURenderer(props);
  await renderer.init();
  return renderer;
}} />
```

- 95% browser coverage (Chrome, Edge, Firefox, Safari 26+)
- TSL (Three Shading Language): write shaders in JS, compiles to WGSL + GLSL
- 2-10x faster for draw-call-heavy scenes; 10-100x for compute

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Import Three.js in server component | Always `dynamic(() => import(...), { ssr: false })` |
| Dim lights after Three.js upgrade | Multiply intensity by `Math.PI` |
| Bloom not working | Set `toneMapped={false}` on glowing materials |
| Events fire on wrong object | Call `e.stopPropagation()` |
| Janky animation speed | Use `delta` parameter in `useFrame`, not fixed increment |
| Memory leak on unmount | Call `.dispose()` on geometries, materials, textures |
| Font crash in Text3D | Convert fonts via facetype.js (JSON format required) |
| R3F v9 type errors | Use `ThreeElements['mesh']` not `MeshProps` |

## Key Resources

- **Docs:** [r3f.docs.pmnd.rs](https://r3f.docs.pmnd.rs/) | [drei.docs.pmnd.rs](https://drei.docs.pmnd.rs/)
- **Course:** [threejs-journey.com](https://threejs-journey.com/) (Bruno Simon)
- **Tutorials:** [tympanus.net/codrops/tag/three-js](https://tympanus.net/codrops/tag/three-js/)
- **Tools:** [gltf.pmnd.rs](https://gltf.pmnd.rs/) (GLTF to JSX) | [polyhaven.com](https://polyhaven.com/) (free HDRI/textures)
- **Starter:** [github.com/pmndrs/react-three-next](https://github.com/pmndrs/react-three-next)
- **Shaders:** [blog.maximeheckel.com](https://blog.maximeheckel.com/) | [threejsroadmap.com](https://threejsroadmap.com/)
- **Inspiration:** [awwwards.com/websites/three-js](https://www.awwwards.com/websites/three-js/)
- **Performance:** [utsubo.com/blog/threejs-best-practices-100-tips](https://www.utsubo.com/blog/threejs-best-practices-100-tips)

See `reference.md` for the complete API reference, Drei component catalog, shader patterns, and detailed code examples.
