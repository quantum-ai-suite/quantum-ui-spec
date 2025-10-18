# QUIML Specification Document

**Version:** 1.1  
**Date:** October 18, 2025  
**Author:** xAI (Grok 4) and Collaborator  
**Status:** Draft  
**Description:** Comprehensive specification for QUIML (Quantum User Interface Markup Language), a declarative, extensible UI framework for cross-platform, AI-driven interfaces. This artifact captures the full design, including architecture, components, styling (updated with hybrid approach), and extensibility. Updated with latest library versions, detailed syntax section, AI integration strategies, build-time compilation for prompts, expanded implementation packages, and hybrid styling for cross-platform consistency. New addition: "library" section for reusable assets like SVG, images, and icons.

## 1. Overview
QUIML (Quantum User Interface Markup Language) is a declarative, extensible UI framework designed for creating futuristic, cross-platform user interfaces. Inspired by quantum-inspired innovation (aligned with "Quantum Co-Op Drive" and "Quantum Watch"), QUIML provides a modern alternative to traditional UI languages like JavaFX FXML, with support for web (React), desktop (Qt/Tauri, JavaFX), and multi-language controllers (JavaScript, Python, C++, Java). It emphasizes modularity, AI integration, and dynamic styling, enabling developers to build responsive, interactive UIs with a professional aesthetic.

### 1.1 Purpose

- Enable rapid UI prototyping and development across platforms.
- Support AI-driven interfaces with real-time data binding and actions.
- Provide a flexible, extensible system for custom components and styles.
- Maintain consistency with a neutral, professional naming convention.

### 1.2 Key Features

- **Cross-Platform:** Compatible with Web, Qt, and JavaFX.
- **Multi-Language:** Supports JavaScript, Python, C++, and Java controllers.
- **Extensible:** Allows user-defined attributes, behaviors, and types.
- **Styling:** Hybrid approach with inline attributes, "style" sections (QUIML-CSS subset), and external CSS for portability.
- **AI-Ready:** Integrates with WebSocket/GraphQL for real-time updates; supports AI-generated prompts for automated design.
- **Asset Library:** New "library" section for defining and referencing reusable assets (e.g., SVGs, images).

## 2. Architecture

### 2.1 Syntax
QUIML uses a YAML-based syntax for its declarative templates, offering a concise and readable format. The root element is `quiml`, with nested structures for layouts, components, and controllers.

Example:
```yaml
quiml: 1.1
controllers:
  - name: MainController
    language: javascript
    script: mainController.js
style: { bg: #000000, color: #00AEEF }  # Hybrid style section
components:
  - Button#btn1:
      text: Click
      style: { bg: #000000, color: #00AEEF }
```

### 2.2 Parser

Implementation: JavaScript-based (e.g., js-yaml v4.1.0) with polyfills for Python (PyYAML v6.0.2), Java (SnakeYAML v2.5).  
Extensibility APIs:  
- `QUIML.registerType(type, definition)`: Adds new component types.  
- `QUIML.registerAttribute(type, name, config)`: Defines new attributes.  
- `QUIML.registerBehavior(type, name, config)`: Adds new event behaviors.  

Runtime: Platform-specific rendering (React v19.2, Qt Widgets, JavaFX Nodes).

### 2.3 Controller System

Multi-Language Support: Controllers are defined with name, language, and script.  
Interface:
```typescript
interface QUIMLController {
  init(): void;
  update(data: any): void;
  destroy(): void;
  [key: string]: (event: any) => void; // Event handlers
}
```

Example:
```yaml
controllers:
  - name: MainController
    language: javascript
    script: mainController.js
```

### 2.4 QUIML File Syntax Details
QUIML files use a relaxed YAML syntax to enhance readability, mimicking JavaScript object literals and CSS declarations. This allows compact inline mappings (e.g., `style: { bg: #000000 }`) without mandatory quoting for hex colors or special characters like `#` in IDs. To ensure compatibility with strict YAML parsers, a pre-processor transforms the relaxed syntax into valid YAML before parsing. This pre-processor handles:

- Expanding inline flow mappings `{ ... }` to block style.
- Quoting values containing `#` (e.g., hex colors) to prevent comment interpretation.
- Quoting keys with special characters (e.g., component IDs like `Button#btn1` becomes `"Button#btn1"`).

The pre-processor can be implemented as a lightweight function (e.g., in JavaScript, Python, or Java) and integrated into the parser pipeline. Example Python implementation:
```python
import re

def expand_flow_mapping(mapping_str, indent_level):
    mapping_str = mapping_str.strip()[1:-1].strip()
    pairs = re.split(r',\s*(?![^{]*})', mapping_str)
    lines = []
    for pair in pairs:
        if ':' in pair:
            key, value = pair.split(':', 1)
            key = key.strip()
            value = value.strip()
            if value.startswith('{'):
                sub_lines = expand_flow_mapping(value, indent_level + 2)
                lines.append(' ' * indent_level + key + ':')
                lines.extend(sub_lines)
                continue
            if '#' in value:
                value = f'"{value}"'
            lines.append(' ' * indent_level + key + ': ' + value)
    return lines

def preprocess_quiml(input_text):
    lines = input_text.split('\n')
    output_lines = []
    for line in lines:
        if re.match(r'^\s*-\s*\w+#\w+:', line):
            line = re.sub(r'(-\s*)(\w+#\w+):', r'\1"\2":', line)
        match = re.match(r'(\s*)([^:]+):\s*\{(.*)\}', line)
        if match:
            indent, key, mapping = match.groups()
            output_lines.append(indent + key + ':')
            expanded = expand_flow_mapping('{' + mapping + '}', len(indent) + 2)
            output_lines.extend(expanded)
        else:
            if ':' in line and not line.strip().startswith('#'):
                parts = line.rsplit(':', 1)
                val_part = parts[1].strip()
                if '#' in val_part:
                    parts[1] = f' "{val_part}"'
                    line = ':'.join(parts)
            output_lines.append(line)
    return '\n'.join(output_lines)
```

#### Root Properties
QUIML files are structured as YAML mappings with the following top-level keys:

| Key         | Description                                                  | Required |
| ----------- | ------------------------------------------------------------ | -------- |
| quiml       | Version (e.g., 1.1).                                         | Yes      |
| controllers | List of controllers.                                         | No       |
| style       | Hybrid style section with QUIML-CSS subset (e.g., :root variables, selectors). | No       |
| styles      | External CSS file(s) for platform-specific styling.          | No       |
| library     | Reusable assets (e.g., SVGs, images) defined once and referenced. | No       |
| components  | List of UI components.                                       | Yes      |

### 2.5 Library Section
The "library" section defines reusable assets like SVGs, images, and icons, which can be referenced in components via `ref: assetID`. This promotes modularity and reduces duplication. Assets are pre-loaded at parse time.

Example:
```yaml
library:
  logoAsset:
    type: Svg
    viewBox: "0 0 50 50"
    elements:
      - Circle: { cx: 25, cy: 25, r: 20, fill: var(--qs-accent) }
  twitterIcon:
    type: Icon
    src: twitter-icon.png
components:
  - Image#logo: { ref: logoAsset }
```

Supported types: Svg (with elements), Image (src/size), Icon (src/size), [Extensible]. References resolve at compile-time for efficiency.

## 3. Components

### 3.1 Controls
Interactive elements.

| Name         | Description           | Key Attributes                             |
| ------------ | --------------------- | ------------------------------------------ |
| Button       | Clickable button.     | text, icon, style, [Extensible]            |
| Label        | Text display.         | text, style, [Extensible]                  |
| TextField    | Input field.          | placeholder, secure, style, [Extensible]   |
| CheckBox     | Toggle checkbox.      | checked, text, style, [Extensible]         |
| RadioButton  | Grouped selection.    | selected, group, text, style, [Extensible] |
| ComboBox     | Dropdown selection.   | items, selected, style, [Extensible]       |
| List         | Scrollable list.      | items, selected, style, [Extensible]       |
| Table        | Data table.           | columns, rows, style, [Extensible]         |
| Tree         | Hierarchical view.    | root, style, [Extensible]                  |
| Image        | Image display.        | src, size, style, ref, [Extensible]        |
| Icon         | Icon display.         | src, size, color, style, ref               |
| Slider       | Value slider.         | min, max, value, style, [Extensible]       |
| ProgressBar  | Progress indicator.   | value, style, [Extensible]                 |
| TabPanel     | Tabbed container.     | tabs, selected, style, [Extensible]        |
| Accordion    | Collapsible sections. | sections, expanded, style, [Extensible]    |
| Card         | Card container.       | content, style, [Extensible]               |
| ToggleButton | Toggle button.        | checked, text, style, [Extensible]         |

### 3.2 Layouts
Structural arrangements for components.

| Name               | Description             | Key Attributes                                               |
| ------------------ | ----------------------- | ------------------------------------------------------------ |
| HorizontalBox      | Horizontal arrangement. | spacing, align, divider, style, [Extensible]                 |
| VerticalBox        | Vertical arrangement.   | spacing, align, divider, style, [Extensible]                 |
| FlowLayout         | Flowing with wrapping.  | spacing, wrap, style, [Extensible]                           |
| BorderLayout       | Fixed regions.          | top, center, bottom, left, right, sticky, style, [Extensible] |
| StackLayout        | Stacked layers.         | components, zIndex, style, [Extensible]                      |
| GridLayout         | Grid arrangement.       | rows, cols, spacing, mode, style, [Extensible]               |
| GridLayoutAdvanced | 2D grid with spans.     | span, style, [Extensible]                                    |

### 3.3 Bindings
Dynamic data and event connections.

| Name          | Description           | Key Attributes               |
| ------------- | --------------------- | ---------------------------- |
| DataBinding   | Links to data source. | source, target, [Extensible] |
| TwoWayBinding | Bidirectional sync.   | source, target, [Extensible] |
| EventBinding  | Triggers on events.   | event, [Extensible]          |

### 3.4 Animations
Visual effects for transitions.

| Name   | Description           | Key Attributes                    |
| ------ | --------------------- | --------------------------------- |
| Jitter | Random offset effect. | duration, infinite, [Extensible]  |
| Pulse  | Scaling effect.       | duration, scale, [Extensible]     |
| Fade   | Opacity transition.   | duration, direction, [Extensible] |
| Glow   | Glowing edge effect.  | duration, intensity, [Extensible] |
| Slide  | Slide transition.     | duration, direction, [Extensible] |

### 3.5 Graphics (New in v1.1)
Vector and asset components, including SVG support based on SVG Tiny 1.2 subset for portability.

| Name             | Description                | Key Attributes                                               |
| ---------------- | -------------------------- | ------------------------------------------------------------ |
| Svg              | Vector graphics container. | width, height, viewBox, elements, src, ref, style, [Extensible] |
| Circle           | Circular shape.            | cx, cy, r, fill, stroke, [Extensible]                        |
| Ellipse          | Oval shape.                | cx, cy, rx, ry, fill, stroke, [Extensible]                   |
| Line             | Line segment.              | x1, y1, x2, y2, stroke, [Extensible]                         |
| Polygon          | Closed polygon.            | points, fill, stroke, [Extensible]                           |
| Polyline         | Open polyline.             | points, stroke, [Extensible]                                 |
| Rect             | Rectangle.                 | x, y, width, height, rx, ry, fill, stroke, [Extensible]      |
| Path             | Custom path.               | d, fill, stroke, [Extensible]                                |
| Text             | Text element.              | x, y, font-size, fill, content, tspans, [Extensible]         |
| Tspan            | Inline text span.          | x, dy, fill, content, [Extensible]                           |
| Animate          | Attribute animation.       | attributeName, from, to, dur, repeatCount, [Extensible]      |
| AnimateColor     | Color animation.           | attributeName, from, to, dur, repeatCount, [Extensible]      |
| AnimateMotion    | Motion along path.         | dur, repeatCount, mpath, [Extensible]                        |
| AnimateTransform | Transform animation.       | type, from, to, dur, repeatCount, [Extensible]               |
| Set              | Set animation.             | attributeName, to, begin, dur, [Extensible]                  |

SVG elements support animations under an `animations` key. Subset excludes scripts, filters, etc., for security/portability.

## 4. Styling Options

### 4.1 Inline Styles

Definition: YAML object in style attribute.  
Properties: bg, color, border, blur, size, pos, anim, hover, [Extensible].  
Example:
```yaml
Button#btn1:
  style: { bg: #000000, color: #00AEEF, blur: 10px }
```

### 4.2 Hybrid "Style" Sections (New in v1.1)
Definition: Top-level "style" key with QUIML-CSS subset (CSS-like syntax, validated for portability). Supports variables (:root), selectors (IDs/classes), and pseudo-selectors (:hover). Merged like HTML <style> tags; imports via separate QUIML files.  
Features: Subset properties (e.g., background-color, font-size); no web-specific (flex/grid) unless mapped.  
Example:
```yaml
style:
  :root { --qs-bg: #000000; }
  Button { background-color: var(--qs-bg); }
  Button:hover { background-color: #333333; }
```

### 4.3 External CSS

Definition: .css file referenced in styles.  
Features: CSS variables (--quiml-*), keyframes, pseudo-classes, fallbacks.  
Example:
```css
:root { --quiml-bg: #000000; }
.quiml-button { background: var(--quiml-bg); }
```

### 4.4 Dynamic Theming

Definition: Controller-driven style updates via JavaScript API.  
API: `QUIML.updateStyle(elementId, styleObject)`.  
Example:
```javascript
QUIML.updateStyle('btn1', { customGlow: '10px' });
```

### 4.5 Platform-Specific Styling

- Web: CSS with style and className.
- Qt: QSS with fallbacks.
- JavaFX: .css with -fx- prefixes.

### 4.6 Extensibility

Mechanism: [Extensible] attributes allow user-defined styles via controllers or type definitions.  
Example:
```javascript
QUIML.registerAttribute('Button', 'customGlow', { default: '0px', apply: (el, v) => el.style.boxShadow = `0 0 ${v} #00AEEF` });
```

## 5. Extensibility

### 5.1 Programmatic Extension

API: registerType, registerAttribute, registerBehavior.  
Example:
```javascript
QUIML.registerAttribute('Button', 'customColor', { default: '#000000', apply: (el, v) => el.style.backgroundColor = v });
```

### 5.2 Type Referencing

Definition: External .quiml-types files for new types.  
Example (custom-types.quiml):
```yaml
- type: CustomButton
  extends: Button
  attributes: { customColor: { default: '#000000' } }
```

## 6. Implementation

### 6.1 Parser

Language: JavaScript (web), Python (Qt), Java (JavaFX).  
Extensibility: Dynamic type/attribute registration.

### 6.2 Rendering

Web: React v19.2 with D3.js v7.9.0, Three.js r172 (latest as of October 2025).  
Qt: QWidgets with QSS.  
JavaFX: Nodes with CSS.

### 6.3 Cross-Platform

Validation: Test on React v19.2, Qt 6.10, JavaFX 25.  
Adaptation: Fallbacks for missing features (e.g., blur).

### 6.4 Implementation Packages
Platform-specific packages for bindings:

| Package Name              | Language/Framework      | Platforms                 |
| ------------------------- | ----------------------- | ------------------------- |
| quantum-ui-java-javafx    | Java/JavaFX             | Desktop/Java environments |
| quantum-ui-js-react       | JavaScript/React        | Web                       |
| quantum-ui-cpp-qt         | C++/Qt                  | Desktop (cross-platform)  |
| quantum-ui-python-pyqt    | Python/PyQt             | Desktop (cross-platform)  |
| quantum-ui-swift-swiftui  | Swift/SwiftUI           | iOS/macOS                 |
| quantum-ui-kotlin-compose | Kotlin/Jetpack Compose  | Android                   |
| quantum-ui-dart-flutter   | Dart/Flutter            | iOS/Android/Web/Desktop   |
| quantum-ui-js-reactnative | JavaScript/React Native | iOS/Android/Web           |
| quantum-ui-csharp-maui    | C#/.NET MAUI            | Windows/Android/iOS/macOS |
| quantum-ui-rust-slint     | Rust/Slint              | Desktop/Mobile/Embedded   |

Core APIs are consistent across packages (e.g., `QUIML.load(filePath)`).

## 7. Security Best Practices
- Use safe YAML parsers to prevent injection.
- Sandbox controllers and validate inputs.
- Sanitize styles and bindings.
- Enforce HTTPS for integrations.
- Regular dependency scans and audits.

## 8. Future Considerations

- AR/VR Support: Extend with WebXR, Qt 3D.
- Performance: Optimize for large datasets (e.g., List, GridLayout).
- Community: Open-source with plugin ecosystem.
- AI Enhancements: Full SudoLang support for controllers; advanced prompt compilation.

End of Specification
