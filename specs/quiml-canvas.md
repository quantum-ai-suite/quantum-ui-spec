# QUIML Canvas Module Specification Document

**Version:** 1.1  
**Date:** October 18, 2025  
**Author:** xAI (Grok 4) and Collaborator  
**Status:** Draft  
**Description:** Specification for the QUIML Canvas Module, an extensible extension providing abstracted 2D/3D graphics support. This module enables declarative primitives, operations, and canvas editing/output, compiling to cross-platform backends (e.g., WebGL/Three.js for web, Skia for mobile/desktop). It organizes into importable sub-modules for modularity. Updated to align with QUIML v1.1, including hybrid styling, new "library" section for reusable assets (e.g., SVGs, images, icons), detailed syntax with pre-processor support, AI integration strategies, and expanded primitives/operations (e.g., animations, gradients). Enhancements focus on asset reusability, cross-platform consistency, and AI-generated graphics.

## 1. Overview
The Canvas Module extends QUIML with a unified "Canvas" component for graphics, supporting direct primitive syntax (e.g., `rect: { ... }`). It abstracts drawing across backends, allowing 2D/3D modes, primitives (lines, spheres), operations (matrices), and dynamic bindings. Designed for projects needing canvas editing/complex output, it's importable as `quantum-ui-module-canvas` (full) or subs (e.g., `.2d`). Updated for QUIML v1.1 compatibility, including hybrid styling (inline + QUIML-CSS sections + external CSS), asset libraries, and AI prompts for automated scene generation.

### 1.1 Purpose
- Provide declarative graphics for 2D/3D UIs with asset reusability.
- Abstract backends for portability (web, mobile, desktop).
- Enable extensible primitives/ops via imports and registrations.
- Support AI-generated content for scenes, shaders, and animations.
- Integrate reusable assets (e.g., SVGs, images) via new "library" section.

### 1.2 Key Features
- **Modes:** 2d/3d abstraction with backend mappings.
- **Primitives:** Direct syntax for shapes (rect, sphere), now including SVG-aligned elements (e.g., Path, Animate).
- **Operations:** Matrix transforms, shaders, gradients, animations.
- **Extensibility:** Register custom ops/types; AI integration for prompt-based generation.
- **Imports:** Full module or subs with dotted hierarchies.
- **Asset Library:** New "library" section for defining/referencing reusable graphics (SVGs, images, icons).
- **Styling:** Hybrid approach for portability, merging inline, sections, and external CSS.
- **AI-Ready:** Supports build-time compilation of AI prompts for dynamic graphics.

## 2. Architecture

### 2.1 Syntax
Canvas uses nested YAML under `drawOps` or `objects` for primitives/ops, with relaxed syntax (e.g., inline mappings `{ ... }`) processed by the v1.1 pre-processor (handles quoting for `#`, expansion of flow mappings). Direct keys (e.g., `rect:`) preferred over `type:`. Modes map to backends at compilation. Supports top-level "library" for reusable assets and "style" for hybrid QUIML-CSS.

Example:
```yaml
quiml: 1.1
import: quantum-ui-module-canvas.3d  # Imports 3D subs unqualified
library:  # New: Reusable assets
  - id: quantum-logo
    type: image
    src: quantum-logo.svg
    size: 50px
  - id: anim-path
    type: svg
    content:
      - Path: { d: "M10 10 L100 100", stroke: #00AEEF }
style:  # Hybrid style section
  :root { --canvas-bg: #000000; --accent: #00AEEF; }
  Canvas { background-color: var(--canvas-bg); }
components:
  - Canvas#viewer:
      mode: 3d
      width: 800px
      height: 600px
      extensions: [high_precision]
      camera: { type: perspective, fov: 75 }
      objects:
        - sphere: { radius: 1, position: [0,0,0], material: basic { color: #00AEEF } }
      bindings:
        - DataBinding: { source: ViewerController.data, target: objects }
      animations:
        - AnimateTransform: { type: rotate, from: [0,0,0], to: [0,360,0], dur: 5s, repeatCount: indefinite }
```

### 2.2 Module Organization
Organized as importable sub-modules with dotted hierarchies. Full import unflattens via aliases if conflicts. Updated with SVG subset integration for reusability.

| Sub-Module | Description                         | Key Primitives/Ops                                           |
| ---------- | ----------------------------------- | ------------------------------------------------------------ |
| vector     | Matrix/path ops for transforms.     | matrix: { translate: [x,y,z] }, path: { moveTo: [x,y], lineTo: [x,y] } |
| 2d         | Flat shapes/fill (SVG-aligned).     | rect: { x,y,w,h,fill }, line: { from,to,width,color }, circle: { cx,cy,r,fill }, path: { d: "M...", fill } |
| 3d         | Volumetric geometries/materials.    | sphere: { radius,segments }, cube: { size }, mesh: { geometry,material } |
| ops        | Shared/advanced (shaders, compute). | applyMatrix: { target: object, matrix: ... }, shader: { vertex: code, fragment: code }, animate: { ... } |
| svg        | SVG subset for vector graphics.     | Path, Rect, Circle, Image, Text, Animate, [as per QUIML v1.1 SVG subset] |

- Full Import: `import: quantum-ui-module-canvas` (all unqualified).
- Selective: `import: quantum-ui-module-canvas.2d` (only 2D ops).

### 2.3 Canvas Component
The core type, abstracted for modes. Updated with hybrid style support and library referencing.

| Attribute  | Type          | Description                                                  | Required     |
| ---------- | ------------- | ------------------------------------------------------------ | ------------ |
| mode       | String        | 2d/3d abstraction (maps to backend).                         | Yes          |
| width      | String/Number | Canvas size (e.g., 100%, 800px).                             | Yes          |
| height     | String/Number | Canvas size.                                                 | Yes          |
| background | String        | Clear color (e.g., #FFFFFF).                                 | No           |
| extensions | List          | Backend extensions (e.g., high_precision).                   | No           |
| camera     | Map           | For 3D (type: perspective/orthographic, fov, etc.).          | No (3D only) |
| lights     | List          | For 3D (point/ambient, color, position).                     | No (3D only) |
| drawOps    | List          | 2D operations (e.g., rect, line).                            | No           |
| objects    | List          | 3D geometries (e.g., sphere, mesh).                          | No (3D only) |
| animations | List          | Animation elements (e.g., Animate, Set).                     | No           |
| bindings   | List          | Dynamic sync (e.g., to controller data).                     | No           |
| event      | Map           | Interactions (e.g., onMouseDrag: controller.rotate).         | No           |
| library    | List          | Local reusable assets (e.g., SVGs, images); refs global if absent. | No           |
| style      | Map/String    | Hybrid styles (inline obj, QUIML-CSS section, or external ref). | No           |

### 2.4 Parser and Pre-Processor
Aligns with QUIML v1.1: Uses js-yaml v4.1.0 (web), PyYAML v6.0.2 (Python), SnakeYAML v2.5 (Java). Pre-processor expands inline mappings, quotes specials (e.g., `#` in colors), ensuring YAML validity.

## 3. Primitives and Operations

### 3.1 Vector Sub-Module
For transforms/paths. Updated with SVG path support.

| Primitive/Op | Description             | Key Attributes                                               |
| ------------ | ----------------------- | ------------------------------------------------------------ |
| matrix       | Transformation matrix.  | translate: [x,y,z], rotate: { axis: y, angle: 45 }, scale: [x,y,z] |
| path         | Vector path for shapes. | moveTo: [x,y], lineTo: [x,y], quadraticCurveTo: [cx,cy,x,y], close: true, d: "M10 10 L100 100" (SVG path data) |

### 3.2 2D Sub-Module
Flat graphics, aligned with SVG subset.

| Primitive/Op | Description     | Key Attributes                                               |
| ------------ | --------------- | ------------------------------------------------------------ |
| rect         | Rectangle.      | x,y,w,h,fill,stroke,radius                                   |
| line         | Line segment.   | from: [x,y], to: [x,y], width, color, dash: [5,5]            |
| circle       | Circle/ellipse. | cx,cy,r,fill,stroke                                          |
| arc          | Arc segment.    | cx,cy,r,startAngle,endAngle,counterclockwise: true           |
| text         | Text rendering. | content: "Hello", position: [x,y], font: { family: "Arial", size: 24, weight: bold, style: italic }, color: #000000, align: center |
| image        | Bitmap image.   | src: "img.png" or library: quantum-logo, position: [x,y], size: [w,h], scale: fit, alpha: 1.0 |
| path         | SVG path.       | d: "M...", fill, stroke, [Extensible]                        |

### 3.3 3D Sub-Module
Volumetric. Updated with animation support.

| Primitive/Op | Description        | Key Attributes                                               |
| ------------ | ------------------ | ------------------------------------------------------------ |
| sphere       | Sphere geometry.   | radius, segments, material: { type: phong, color: #00AEEF }  |
| cube         | Cube/box.          | size: [w,h,d], position: [x,y,z]                             |
| mesh         | Custom mesh.       | geometry: custom, material: shader { vertex: code, fragment: code } |
| model        | Loaded model.      | file: model.glb, scale: 1, position: [x,y,z]                 |
| text3d       | 3D text extrusion. | content: "Hello", font: { family: "Arial", size: 1 }, extrude: 0.1, color: #000000 |

### 3.4 Ops Sub-Module
Shared/advanced, expanded with gradients, patterns, clips, and animations from SVG subset.

| Primitive/Op     | Description            | Key Attributes                                               |
| ---------------- | ---------------------- | ------------------------------------------------------------ |
| applyMatrix      | Apply transform.       | target: object, matrix: ...                                  |
| shader           | Custom shader.         | vertex: code, fragment: code, uniforms: { time: 0.0 }        |
| linearGradient   | Linear color gradient. | from: [x1,y1], to: [x2,y2], stops: [{ offset: 0, color: #FF0000 }, { offset: 1, color: #00FF00 }] |
| radialGradient   | Radial color gradient. | center: [cx,cy], radius: r, stops: [...]                     |
| pattern          | Repeating pattern.     | src: "pattern.png" or library: id, repeat: xy, scale: [sx,sy] |
| clip             | Clipping region.       | path: ..., mode: intersect                                   |
| animate          | Attribute animation.   | attributeName, from, to, dur, repeatCount, [Extensible]      |
| animateColor     | Color animation.       | attributeName, from, to, dur, repeatCount, [Extensible]      |
| animateMotion    | Motion along path.     | dur, repeatCount, mpath: path-ref, [Extensible]              |
| animateTransform | Transform animation.   | type: rotate/scale/etc., from, to, dur, repeatCount, [Extensible] |
| set              | Set animation.         | attributeName, to, begin, dur, [Extensible]                  |

## 4. Styling Options
Aligns with QUIML v1.1 hybrid styling: Inline (e.g., `style: { fill: #00AEEF }`), top-level "style" sections (QUIML-CSS with variables/selectors), external CSS refs. Dynamic updates via `QUIML.updateStyle`. Platform-specific (e.g., CSS for web, QSS for Qt).

## 5. Extensibility
- Register custom ops/types: `QUIML.registerOp('customPath', impl)` (backends map to native).
- AI prompts generate primitives (e.g., "Add sphere with matrix rotate"); build-time compilation for prompts.

## 6. Implementation
- Backends: Web (WebGL/Three.js r172), Mobile (Skia in Flutter/Compose), Desktop (QPainter in Qt), updated with D3.js v7.9.0.
- Compilation: YAML ops to code (e.g., Three.js Mesh for sphere); integrates with v1.1 packages (e.g., quantum-ui-js-react v19.2).

End of Specification