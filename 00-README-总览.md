# HT for Web 与 Three.js 实现细节对比

> 对比对象：**HT for Web** 与 **Three.js r125**（`three-0.125.0/package/src/`）。
> 记录日期：2026-08-26。

## 结论

HT 和 Three.js 不是同一层级的产品。

HT 是面向工业组态和拓扑可视化的业务框架：`DataModel` 保存业务数据，`Node`、`Edge`、`Group`、`Shape` 是可序列化的业务图元，`GraphView` 和 `Graph3dView` 读取同一份数据分别输出 Canvas 2D 拓扑和 WebGL 3D 场景，`ht.widget` 提供表格、树、属性面板和工具栏。

Three.js 是通用 3D 渲染库：`Scene`/`Object3D` 组织渲染对象，`Mesh` 连接几何体与材质，`WebGLRenderer` 提交绘制命令。业务数据、2D 拓扑、组件、序列化和业务动画都由应用自己补齐。

两者真正的深度交集在 WebGL shader：HT 的 PBR、Phong、GGX/Smith/Schlick、多散射、cubeUV/PMREM、FXAA 和部分 Bloom/景深代码与 Three.js 同源。代码指纹把相关 HT shader 定位到 Three.js r133～r136；本文为了可复现，统一使用 r125 展开对比，并在 01、07、09 章标出版本差异。

## 对比基线的选取

| | 选取的基线 | 说明 |
|---|---|---|
| HT 侧 | HT for Web 的业务演示工程（泵站、储能电站两个 2D + 3D 案例） | 以工程内 `src/**` 的实现为准，公开 API 名称与 API 文档一致 |
| Three.js 侧 | r125 源码 `three-0.125.0/package/src/**` | 之所以取 r125，是因为它与 HT 的 shader 代码有可逐函数比对的交集；r125 之外的版本用于佐证同源区间 |

两侧对比的原则：**比实现，不比营销**。凡是可以逐函数、逐方法对照的地方，都给出对应的类名/方法名，而不是停留在功能描述。

## HT 侧的实现组织

HT 的实现按功能域集中：核心（命名空间、默认配置、资源与序列化）、数学、动画、几何、渲染底层（WebGL 程序、shader、材质、环境贴图、阴影）、Canvas 2D 绘制、图模型（Data/Node/Edge/Group 与 GraphView）、3D 图形（Graph3dView、渲染列表、模型、交互、拾取、后处理）、组件（表单、树、列表、属性面板、工具栏）。

| 功能域 | 内容 |
|---|---|
| 核心 | `ht` 命名空间、默认配置、资源、序列化、心跳 |
| 数学 | Vector、Matrix、Quaternion、Euler、包围体、插值器、缓动 |
| 动画 | 时间轴、轨道、片段、绑定与全局调度 |
| 几何 | 路径、挤出和几何曲面 |
| 渲染底层 | WebGL 程序、shader 库、材质、环境贴图和阴影 |
| 2D 画布 | Canvas 2D 绘制工具和绘制状态 |
| 图模型 | Data、Node、Edge、Group、GraphView 和拓扑交互 |
| 3D 图形 | Graph3dView、渲染列表、模型、交互、拾取和后处理 |
| 组件 | 表单、树、列表、属性面板、工具栏和图标 |

HT 的类和函数通过 ES module import/export 组织；对外则是传统全局脚本入口，公开类挂在 `ht` 命名空间下（`ht.Data`、`ht.graph.GraphView`、`ht.widget.*` 等）。

## Three.js r125 的实现组织

```text
three-0.125.0/package/src/
├─ core/、scenes/、objects/       场景对象与渲染树
├─ geometries/、materials/        几何体与材质
├─ cameras/、lights/              相机与灯光
├─ renderers/                    WebGLRenderer 与 WebGL 管线
├─ math/                         Vector、Matrix、Quaternion、Euler
├─ loaders/、extras/             加载器和 PMREM 工具
└─ animation/                    AnimationClip、Mixer、Action
```

核心工作流是 `new THREE.Scene()` → `scene.add(mesh)` → `renderer.render(scene, camera)`。Object3D 树就是渲染结构本身；`userData` 只是一个开放对象，不会自动触发业务绑定、序列化或重绘。

## 能力对照

| 领域 | HT | Three.js r125 |
|---|---|---|
| 定位 | 工业组态/拓扑业务框架 | 通用 3D 渲染库 |
| 场景组织 | `DataModel` + `Data` 父子关系，2D/3D 共用 | `Scene` + `Object3D` 渲染树 |
| 属性 | `s()` 样式和 `a()` 业务属性，变更自动通知 | 对象属性或 `userData`，联动由应用实现 |
| 2D 拓扑 | GraphView、节点、连线、路由、编辑、组件完整 | 无 2D 拓扑语义 |
| 3D 调度 | 数据失效、缓存、渲染列表、批处理和按需重绘 | 应用驱动 RAF，渲染器遍历场景 |
| 材质 | `shape3d.*` 样式映射到 uniform，无独立材质对象 | MeshStandard/PhysicalMaterial 等独立实例 |
| 半透明 | 内建 OIT 和渲染层 | 默认透明排序 |
| 阴影 | 场景级单套 shadow map | 每个启用阴影的光源独立 shadow map |
| 环境贴图 | PMREM、skybox、probe 和场景属性一体化 | PMREMGenerator 等工具由应用组合 |
| 拾取 | GPU 颜色 pass，读像素反查数据对象 | Raycaster CPU 几何求交 |
| 模型加载 | 名称注册表和 URL/modelType 分派 | loader 实例和开放生态 |
| 动画 | `startAnim` 业务动画 + 时间轴/轨道/绑定体系 | AnimationClip/Mixer/Action 关键帧系统 |
| 扩展 | 全局注册表 + 独立 script 插件 | import 类 + examples/jsm 组合 |
| 序列化 | 工程级 `{v,p,a,d}` 数据格式 | 场景对象 `toJSON()` |

## 两个案例的回归边界

文档涉及的两个案例（泵站与储能电站）都只加载构建后的 HT 全局入口。

储能电站的三维场景只有 `MapView`、`index`、`OutDoor3d`、`StorageTank3d` 四个 key；`Loading` 是加载页，不是三维场景。储能电站资源多，浏览器对比一律使用 800×600 小窗口，避免把大窗口的渲染压力误判成行为问题。

## 章节导航

| 章节 | 内容 |
|---|---|
| [01 架构总览与版本同源性](01-架构总览与版本同源性.md) | 命名空间、模块入口、类层级和 shader 版本指纹 |
| [02 场景图与数据模型](02-场景图与数据模型.md) | DataModel/Data/Node 与 Scene/Object3D |
| [03 数学库](03-数学库.md) | Vector、Matrix、Quaternion、Euler、Easing 的同源与差异 |
| [04 2D 拓扑渲染与组件](04-2D拓扑渲染与组件.md) | GraphView、连线、路由和 widget |
| [05 3D 渲染器架构与渲染管线](05-3D渲染器架构与渲染管线.md) | 渲染循环、列表、batch、OIT、UBO、拾取 |
| [06 材质系统与样式体系](06-材质系统与样式体系.md) | shape3d 样式和 Three 材质对象 |
| [07 着色器与光照模型](07-着色器与光照模型.md) | shader chunk、PBR、Phong、光照和同源代码 |
| [08 阴影系统](08-阴影系统.md) | shadow map 全链路 |
| [09 环境贴图与天空盒](09-环境贴图与天空盒.md) | PMREM、probe 和 skybox |
| [10 后处理](10-后处理.md) | Bloom、DOF、FXAA、LUT 和色调映射 |
| [11 相机交互与拾取](11-相机交互与拾取.md) | 相机、交互器、GPU/CPU 拾取 |
| [12 几何体与模型加载](12-几何体与模型加载.md) | shape3d 几何、注册模型和外部 loader |
| [13 动画与业务特性](13-动画与业务特性.md) | startAnim、绑定、流动、热力图和选中 |
| [14 动画时间轴与属性绑定](14-动画时间轴与属性绑定.md) | 时间轴、轨道、绑定规则、状态绑定和缓动 |
| [15 插件体系与扩展机制](15-插件体系与扩展机制.md) | 全局注册表与 Three.js 的扩展方式对比 |
| [16 对比口径与边界说明](16-对比口径与边界说明.md) | 对比原则、可比范围与常见误读 |