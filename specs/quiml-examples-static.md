# QUIML Static Page Examples

QUIML supports static page generation for landing pages, dashboards, and more, with hybrid styling for cross-platform consistency. Below is an example of a Quantum Suite-themed landing page using the hybrid style approach (no external CSS file). The `style` section uses a validated QUIML-CSS subset, including hover pseudo-selectors for interactive visuals (translated where supported, e.g., web `:hover` or JavaFX mouse events).

This example focuses on visuals only—no controllers or events.

```yaml
quiml: 1.1
style:  # Hybrid style section: QUIML-CSS subset for cross-platform
  :root { --qs-bg: #000000; --qs-text: #FFFFFF; --qs-accent: #00AEEF; --qs-secondary: #333333; --qs-font: Arial, sans-serif; --qs-hover: #008BBF; }
  #mainLayout { background-color: var(--qs-bg); color: var(--qs-text); font-family: var(--qs-font); padding: 20px; }
  #header { background-color: var(--qs-secondary); padding: 10px; border-bottom: 2px solid var(--qs-accent); }
  #logo { width: 50px; height: 50px; }
  #navbar Label { cursor: pointer; color: var(--qs-text); }
  #navbar Label:hover { color: var(--qs-hover); }
  #search { background-color: var(--qs-bg); color: var(--qs-text); border: 1px solid var(--qs-accent); padding: 5px; border-radius: 5px; }
  #introSection { text-align: center; padding: 50px 0; }
  #introTitle { font-size: 36px; color: var(--qs-accent); }
  #introDesc { font-size: 18px; }
  #actionButtons Button { background-color: var(--qs-accent); color: var(--qs-bg); padding: 10px 20px; border-radius: 5px; cursor: pointer; }
  #actionButtons Button:hover { background-color: var(--qs-hover); }
  #productsSection, #servicesSection { padding: 40px 0; justify-content: center; }
  .card { background-color: var(--qs-secondary); padding: 20px; border-radius: 10px; width: 300px; text-align: center; }
  .card:hover { background-color: #444444; }
  .card Label:first-child { font-size: 24px; color: var(--qs-accent); }
  .card Label:last-child { font-size: 16px; }
  #footer { background-color: var(--qs-secondary); padding: 20px; text-align: center; border-top: 2px solid var(--qs-accent); }
  #contactLinks Label, #navTrees Label { color: var(--qs-text); font-size: 14px; }
  #contactLinks Label:hover, #navTrees Label:hover { color: var(--qs-hover); }
  .icon { width: 30px; height: 30px; cursor: pointer; }
  .icon:hover { opacity: 0.8; }
  #copyright, #version { font-size: 12px; color: var(--qs-text); }
components:
  - VerticalBox#mainLayout:
      components:
        - HorizontalBox#header:
            spacing: 20
            align: center
            components:
              - Image#logo:
                  src: quantum-logo.png
                  size: 50px
              - HorizontalBox#navbar:
                  spacing: 30
                  components:
                    - Label#navHome: { text: Home }
                    - Label#navProducts: { text: Products }
                    - Label#navServices: { text: Services }
                    - Label#navAbout: { text: About }
                    - Label#navContact: { text: Contact }
              - TextField#search:
                  placeholder: Search...
        - VerticalBox#introSection:
            spacing: 20
            align: center
            components:
              - Label#introTitle: { text: Welcome to Quantum Suite }
              - Label#introDesc: { text: Revolutionize your business with AI-driven tools. }
              - HorizontalBox#actionButtons:
                  spacing: 15
                  components:
                    - Button#getStarted: { text: Get Started }
                    - Button#learnMore: { text: Learn More }
        - HorizontalBox#productsSection:
            spacing: 40
            components:
              - Card#productQBOS:
                  content:
                    - Label#qbosTitle: { text: QBOS - Business OS }
                    - Label#qbosDesc: { text: Unified ERP, CRM, and more with self-refining AI. }
              - Card#productQW:
                  content:
                    - Label#qwTitle: { text: Quantum Watch - NPM }
                    - Label#qwDesc: { text: High-performance network monitoring and analytics. }
        - HorizontalBox#servicesSection:
            spacing: 40
            components:
              - Card#serviceConsulting:
                  content:
                    - Label#consultTitle: { text: AI Consulting }
                    - Label#consultDesc: { text: Tailored AI integrations for your enterprise. }
              - Card#serviceSupport:
                  content:
                    - Label#supportTitle: { text: 24/7 Support }
                    - Label#supportDesc: { text: Dedicated help for Quantum Suite products. }
        - VerticalBox#footer:
            spacing: 20
            align: center
            components:
              - HorizontalBox#contactLinks:
                  spacing: 20
                  components:
                    - Label#email: { text: info@quantumsuite.com }
                    - Label#phone: { text: +1-800-QUANTUM }
              - HorizontalBox#socialMedia:
                  spacing: 20
                  components:
                    - Icon#twitter: { src: twitter-icon.png }
                    - Icon#linkedin: { src: linkedin-icon.png }
                    - Icon#facebook: { src: facebook-icon.png }
              - HorizontalBox#navTrees:
                  spacing: 30
                  components:
                    - VerticalBox#companyNav:
                        components:
                          - Label#company: { text: Company }
                          - Label#about: { text: About Us }
                          - Label#careers: { text: Careers }
                    - VerticalBox#legalNav:
                        components:
                          - Label#legal: { text: Legal }
                          - Label#privacy: { text: Privacy Policy }
                          - Label#terms: { text: Terms of Service }
              - Label#copyright: { text: © 2025 Quantum Suite. All rights reserved. }
              - Label#version: { text: Version 1.0 }
~~~

## Notes

- **Style Section**: Contains validated QUIML-CSS for portability (e.g., hover effects map to platform events).
- **Compilation**: Translates to web (React with CSS), JavaFX (nodes with loaded CSS), Qt (QML with QSS), etc., ensuring Quantum Suite's dark theme consistency.
- **Hover**: `:hover` selectors enhance interactivity (e.g., button color change); emulated in non-web targets via event listeners if needed.

End of Artifact
