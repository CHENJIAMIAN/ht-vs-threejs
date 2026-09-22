# HT for Web vs. Three.js: A Detailed Implementation Comparison

[中文](README.md) | **English**

> Taking the HT for Web business-visualization framework apart layer by layer and putting it next to Three.js: which parts share a design, which merely look alike, and which are not even on the same level.

## What this is

A 16-chapter analysis comparing HT for Web and Three.js function domain by function domain. The HT side is an HT for Web business demo project (a pump station and an energy-storage plant, each 2D + 3D); the Three.js side is the r125 source tree.

The repository contains Markdown analysis only. It ships no third-party source code or distribution packages.

## Three-sentence conclusion

1. **Different product tiers.** HT is a framework where one data model feeds a 2D topology view and a 3D scene at the same time; Three.js is a general-purpose library of composable render objects. An HT `Node` is not a `Mesh`.
2. **The real shared lineage is in shaders and math.** HT's PBR / GGX / multi-scattering / cubeUV / PMREM / FXAA code is derived from Three.js, with fingerprints landing on **r133–r136** (not r125 itself); large parts of the math library can be compared method by method.
3. **Control ownership is inverted.** Three.js hands "when to render" and "when to advance animation" to the application (`renderer.render` / `mixer.update`); HT pulls both into the core — invalidated data triggers a redraw, and a global heartbeat drives animation. That is the biggest architectural difference.

## Chapter index

Document bodies are written in Chinese; the file names below are the actual chapter files.

| Chapter | Content |
|---|---|
| [00 Overview](00-README-总览.md) | Baselines, implementation layout, capability matrix |
| [01 Architecture and version lineage](01-架构总览与版本同源性.md) | Namespaces, module entry, class hierarchy, shader version fingerprints |
| [02 Scene graph and data model](02-场景图与数据模型.md) | DataModel/Data/Node vs. Scene/Object3D, notifiers |
| [03 Math library](03-数学库.md) | Vector, Matrix, Quaternion, Euler, Line3, Easing |
| [04 2D topology rendering and widgets](04-2D拓扑渲染与组件.md) | GraphView, edges, routing, widget set |
| [05 3D renderer architecture and pipeline](05-3D渲染器架构与渲染管线.md) | Render loop, render lists, batching, OIT, UBO, heartbeat, fps limiting |
| [06 Materials and styling](06-材质系统与样式体系.md) | shape3d styles vs. Three.js material objects |
| [07 Shaders and lighting model](07-着色器与光照模型.md) | Shader chunks, PBR, Phong, lighting, shared source |
| [08 Shadow system](08-阴影系统.md) | Scene-level shadow map, end to end |
| [09 Environment maps and skybox](09-环境贴图与天空盒.md) | PMREM, probes, skybox |
| [10 Post-processing](10-后处理.md) | Bloom, DOF, FXAA, LUT, tone mapping |
| [11 Camera, interaction and picking](11-相机交互与拾取.md) | Cameras, interactors, GPU/CPU picking, damping |
| [12 Geometry and model loading](12-几何体与模型加载.md) | shape3d geometry, registered models, external loaders |
| [13 Animation and business features](13-动画与业务特性.md) | startAnim, binding, flow, heatmaps, selection |
| [14 Animation timeline and property binding](14-动画时间轴与属性绑定.md) | Timeline, tracks, binding rules, state binding, easing |
| [15 Plugin system and extension mechanism](15-插件体系与扩展机制.md) | Global registry vs. Three.js extension style |
| [16 Scope and boundaries of this comparison](16-对比口径与边界说明.md) | Judging principles, comparable scope, common misreadings |

## Baselines

| | Content |
|---|---|
| HT side | An HT for Web business demo project: a pump station and an energy-storage plant covering 2D topology and 3D scenes; class and method names match the public API documentation |
| Three.js side | r125 source at `three-0.125.0/package/src/**`; other versions are used to corroborate the shared shader range |

## How to read

- Want the conclusion first: see the capability matrix in [00 Overview](00-README-总览.md).
- Looking for a specific subsystem: jump straight to the chapter; every chapter ends with a comparison table.
- Want the boundaries and common misreadings: read [chapter 16](16-对比口径与边界说明.md).

## Notes

- The chapters are written for engineering use: each gives concrete differences plus migration caveats, not abstract concept tours.
- Three.js references always keep the `three-0.125.0/package/src/**` prefix so they can be located in the source tree.
- HT for Web is a commercial product of HT (Hightopo). This repository is technical analysis only and ships none of its source or licensing.