### Detailed QUIML Mobile Examples (Static)

QUIML's declarative nature makes it ideal for mobile development, where responsive layouts and performance are key. With the updated spec (v1.1), these examples focus on static visuals only—no controllers, bindings, or events. Styles use the hybrid "style" section with QUIML-CSS subset for cross-platform consistency. Examples are optimized for vertical scrolling, safe areas, and dark/light modes, compiling to targets like SwiftUI (iOS), Jetpack Compose (Android), Flutter, or React Native.

## 1. Simple Login Screen (Basic Form with Responsiveness)
This example creates a centered login form with input fields and a submit button. It's mobile-optimized with full-screen vertical layout.

**QUIML YAML:**
```yaml
quiml: 1.1
style:  # Hybrid style section
  :root { --qs-bg: #000000; --qs-text: #FFFFFF; --qs-accent: #00AEEF; --qs-secondary: #333333; --qs-hover: #008BBF; }
  #mainLayout { background-color: var(--qs-bg); color: var(--qs-text); padding: 40px; }
  #title { font-size: 24px; color: var(--qs-accent); font-weight: bold; }
  .input { background-color: var(--qs-secondary); color: var(--qs-text); border-radius: 8px; padding: 10px; }
  #submit { background-color: var(--qs-accent); color: var(--qs-bg); border-radius: 8px; padding: 15px; }
  #submit:hover { background-color: var(--qs-hover); }
  #forgot { color: var(--qs-accent); font-size: 14px; }
  #forgot:hover { color: var(--qs-hover); }
components:
  - VerticalBox#mainLayout:
      spacing: 20
      align: center
      components:
        - Label#title:
            text: "Login to Quantum App"
        - TextField#username:
            placeholder: "Username"
            class: input
        - TextField#password:
            placeholder: "Password"
            secure: true
            class: input
        - Button#submit:
            text: "Login"
        - Label#forgot:
            text: "Forgot Password?"
```

**Key Features and Mobile Optimizations:**
- **Layout:** `VerticalBox` ensures vertical stacking for mobile scrolling; safe areas handled via platform-specific fallbacks.
- **Inputs:** `TextField` with `secure: true` for passwords.
- **Styles:** Hybrid section with hover for buttons/links (emulated on mobile via touch events).

## 2. Scrollable List View (Data-Driven Mobile Feed)
A mobile-optimized list (e.g., social feed) with static items.

**QUIML YAML:**
```yaml
quiml: 1.1
style:  # Hybrid style section
  :root { --qs-bg: #000000; --qs-text: #FFFFFF; --qs-accent: #00AEEF; --qs-secondary: #333333; --qs-hover: #008BBF; }
  #mainLayout { background-color: var(--qs-bg); color: var(--qs-text); }
  #feedList { item-height: 80px; background-color: #111111; }
  .itemBox { padding: 10px; border-bottom: 1px solid #333333; }
  .itemBox:hover { background-color: #222222; }
  #itemIcon { width: 60px; height: 60px; }
  #itemText { color: var(--qs-accent); }
  #itemSeparator { orientation: horizontal; color: #333333; }
components:
  - VerticalBox#mainLayout:
      components:
        - List#feedList:
            items: [ { title: "Item 1", image: "img1.png" }, { title: "Item 2", image: "img2.png" } ]  # Static items
            itemTemplate:
              - HorizontalBox#itemBox:
                  class: itemBox
                  components:
                    - Icon#itemIcon:
                        src: "{item.image}"
                        size: 60px
                    - Label#itemText:
                        text: "{item.title}"
                    - Divider#itemSeparator:
                        orientation: horizontal
```

**Key Features and Mobile Optimizations:**
- **List Handling:** `List` with `itemTemplate` for reusable cells; supports virtualization for long lists (mobile perf).
- **Styles:** Hover on items for touch feedback (emulated on mobile).

## 3. Tabbed Navigation with Bottom Bar (Mobile App Shell)
A bottom-tabbed navigation for a multi-view app.

**QUIML YAML:**
```yaml
quiml: 1.1
style:  # Hybrid style section
  :root { --qs-bg: #000000; --qs-text: #FFFFFF; --qs-accent: #00AEEF; --qs-secondary: #333333; --qs-hover: #008BBF; }
  #tabNav { position: bottom; background-color: var(--qs-bg); }
  #tabNav Tab { color: var(--qs-text); }
  #tabNav Tab:hover { color: var(--qs-hover); }
  .card { background-color: var(--qs-secondary); padding: 20px; border-radius: 10px; }
  .card:hover { background-color: #444444; }
  #darkMode { color: var(--qs-text); }
components:
  - TabPanel#tabNav:
      selected: 0
      tabs:
        - title: "Home"
          icon: home.svg
          content:
            - Label#homeLabel:
                text: "Welcome Home"
        - title: "Profile"
          icon: profile.svg
          content:
            - Card#profileCard:
                content:
                  - TextField#name:
                      text: "User Name"
        - title: "Settings"
          icon: settings.svg
          content:
            - ToggleButton#darkMode:
                text: "Dark Mode"
                checked: true
```

**Key Features and Mobile Optimizations:**
- **Navigation:** `TabPanel` with `position: bottom` for mobile tabs; icons for touch-friendly UI.
- **Styles:** Hover on tabs for feedback.

These examples showcase QUIML's flexibility for mobile, with hybrid styles enabling rapid iteration. For production, compile via `quiml build --target=swift-swiftui` to generate platform code.