---
layout: full
---

<PresentationHub />

<style>
:root {
  --ap-paper: #f4f0e8;
  --ap-paper-soft: #faf7f1;
  --ap-grid: rgba(126, 93, 67, 0.1);
  --ap-dot: rgba(116, 88, 68, 0.18);
  --ap-brown: #7c2d12;
  --ap-brown-soft: #a6653a;
  --ap-blue: #1f6f95;
  --ap-blue-soft: #dce7eb;
  --ap-line: #d8cabd;
  --ap-muted: #8d7b6d;
  --ap-text: #4f3e32;
  --ap-panel: rgba(250, 247, 241, 0.88);
  --slidev-controls-foreground: var(--ap-text);
}

.slidev-glass-effect,
.bg-main {
  background-color: rgba(250, 247, 241, 0.88) !important;
}

[class*="z-nav"] .bg-main,
[class*="z-nav"] .slidev-glass-effect {
  border: 1px solid var(--ap-line) !important;
  border-radius: 6px !important;
  color: var(--ap-text) !important;
  box-shadow: 0 14px 30px rgba(92, 67, 49, 0.12) !important;
  backdrop-filter: blur(8px);
}

[class*="z-nav"] button,
[class*="z-nav"] [role="button"] {
  color: var(--ap-text) !important;
}

[class*="z-nav"] button:hover,
[class*="z-nav"] [role="button"]:hover {
  color: var(--ap-blue) !important;
}

nav.flex.flex-col > div.bg-main,
nav.flex.flex-col > div.slidev-glass-effect,
nav.flex.flex-col > div[class*="bg-main"],
nav.flex.flex-col > div[class*="slidev-glass-effect"] {
  background: rgba(250, 247, 241, 0.92) !important;
  border: 1px solid var(--ap-line) !important;
  border-radius: 6px !important;
  color: var(--ap-text) !important;
  box-shadow: 0 14px 30px rgba(92, 67, 49, 0.12) !important;
  backdrop-filter: blur(8px);
}

nav.flex.flex-col > div.bg-main *,
nav.flex.flex-col > div.slidev-glass-effect *,
nav.flex.flex-col > div[class*="bg-main"] *,
nav.flex.flex-col > div[class*="slidev-glass-effect"] * {
  color: var(--ap-text) !important;
}

:global(nav.flex.flex-col > div.bg-main),
:global(nav.flex.flex-col > div.slidev-glass-effect),
:global(nav.flex.flex-col > div[class*="bg-main"]),
:global(nav.flex.flex-col > div[class*="slidev-glass-effect"]) {
  background: rgba(250, 247, 241, 0.92) !important;
  border: 1px solid #d8cabd !important;
  border-radius: 6px !important;
  color: #4f3e32 !important;
  box-shadow: 0 14px 30px rgba(92, 67, 49, 0.12) !important;
  backdrop-filter: blur(8px);
}

:global(nav.flex.flex-col > div.bg-main *),
:global(nav.flex.flex-col > div.slidev-glass-effect *),
:global(nav.flex.flex-col > div[class*="bg-main"] *),
:global(nav.flex.flex-col > div[class*="slidev-glass-effect"] *) {
  color: #4f3e32 !important;
}

.title-cover {
  position: fixed;
  inset: 0;
  display: grid;
  align-content: start;
  justify-items: start;
  gap: 28px;
  padding: clamp(70px, 10vh, 110px) clamp(32px, 5vw, 70px);
  color: var(--ap-brown);
  background:
    linear-gradient(rgba(126, 93, 67, 0.075) 1px, transparent 1px),
    linear-gradient(90deg, rgba(126, 93, 67, 0.075) 1px, transparent 1px),
    radial-gradient(circle, var(--ap-dot) 1px, transparent 1.4px),
    linear-gradient(180deg, #f8f4ed 0%, #eee8df 100%);
  background-size: 72px 72px, 72px 72px, 24px 24px, auto;
  overflow: hidden;
  font-family: "Cascadia Code", "JetBrains Mono", "Fira Code", Consolas, ui-monospace, monospace;
}

.title-cover::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(244, 240, 232, 0), rgba(230, 235, 233, 0.78));
  pointer-events: none;
}

.cover-eyebrow,
.title-cover h1,
.title-cover p,
.cover-flow {
  position: relative;
}

.cover-eyebrow {
  color: var(--ap-brown);
  font-size: 16px;
  font-weight: 900;
  letter-spacing: 0.32em;
  text-transform: uppercase;
}

.title-cover h1 {
  max-width: 920px;
  margin: 0;
  font-size: clamp(34px, 4vw, 58px);
  line-height: 1.05;
  letter-spacing: 0;
  text-align: left;
  font-weight: 900;
  text-shadow: 0 2px 0 rgba(255, 255, 255, 0.72);
}

.title-cover p {
  max-width: 850px;
  margin: 0;
  color: var(--ap-text);
  font-size: clamp(18px, 1.8vw, 25px);
  font-weight: 700;
  line-height: 1.45;
}

.cover-flow {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 12px;
  max-width: 980px;
  margin-top: 10px;
}

.cover-flow span {
  padding: 11px 14px;
  border: 1px solid var(--ap-line);
  border-radius: 999px;
  color: var(--ap-brown);
  background: rgba(250, 247, 241, 0.86);
  font-size: 15px;
  font-weight: 900;
  text-transform: uppercase;
  box-shadow: 0 10px 22px rgba(92, 67, 49, 0.08);
}

.cover-flow i {
  width: 34px;
  height: 2px;
  background: var(--ap-blue);
}

.ap-slide {
  position: fixed;
  inset: 0;
  display: grid;
  align-content: center;
  gap: 32px;
  padding: clamp(52px, 7vh, 82px) clamp(34px, 6vw, 86px);
  color: var(--ap-text);
  background:
    linear-gradient(rgba(126, 93, 67, 0.07) 1px, transparent 1px),
    linear-gradient(90deg, rgba(126, 93, 67, 0.07) 1px, transparent 1px),
    radial-gradient(circle, var(--ap-dot) 1px, transparent 1.4px),
    linear-gradient(180deg, #f8f4ed 0%, #eee8df 100%);
  background-size: 72px 72px, 72px 72px, 24px 24px, auto;
  overflow: hidden;
  font-family: "Cascadia Code", "JetBrains Mono", "Fira Code", Consolas, ui-monospace, monospace;
}

.ap-slide::before {
  content: "";
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(244, 240, 232, 0), rgba(227, 234, 232, 0.62));
  pointer-events: none;
}

.ap-slide > * {
  position: relative;
}

.ap-kicker {
  color: var(--ap-brown);
  font-size: 14px;
  font-weight: 900;
  letter-spacing: 0.22em;
  text-transform: uppercase;
}

.ap-slide h2 {
  max-width: 1040px;
  margin: 0;
  color: var(--ap-brown);
  font-size: clamp(32px, 4.2vw, 58px);
  font-weight: 900;
  line-height: 1.05;
  letter-spacing: 0;
}

.ap-slide p {
  max-width: 1040px;
  margin: 0;
  color: var(--ap-text);
  font-size: clamp(19px, 1.9vw, 26px);
  font-weight: 700;
  line-height: 1.45;
}

.ap-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 18px;
}

.ap-card,
.ap-step,
.ap-role,
.ap-metric {
  min-height: 150px;
  padding: 22px;
  border: 1px solid var(--ap-line);
  border-radius: 6px;
  background: var(--ap-panel);
  box-shadow: 0 14px 30px rgba(92, 67, 49, 0.08);
}

.ap-card strong,
.ap-step strong,
.ap-role strong,
.ap-metric strong {
  display: block;
  margin-bottom: 12px;
  color: var(--ap-brown);
  font-size: 20px;
  font-weight: 900;
  line-height: 1.15;
}

.ap-card span,
.ap-step span,
.ap-role span,
.ap-metric span {
  color: var(--ap-text);
  font-size: 16px;
  font-weight: 700;
  line-height: 1.35;
}

.ap-two {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 22px;
}

.ap-list {
  display: grid;
  gap: 14px;
  margin: 0;
  padding: 0;
  list-style: none;
}

.ap-list li {
  padding: 16px 18px;
  border-left: 4px solid var(--ap-blue);
  border-radius: 4px;
  color: var(--ap-text);
  background: rgba(250, 247, 241, 0.88);
  box-shadow: 0 10px 22px rgba(92, 67, 49, 0.07);
  font-size: 19px;
  font-weight: 800;
  line-height: 1.3;
}

.ap-flowline {
  display: grid;
  grid-template-columns: repeat(5, minmax(0, 1fr));
  gap: 14px;
}

.ap-step {
  min-height: 170px;
}

.ap-step small {
  display: block;
  margin-bottom: 12px;
  color: var(--ap-blue);
  font-size: 13px;
  font-weight: 900;
  letter-spacing: 0.18em;
  text-transform: uppercase;
}

.ap-wide {
  grid-column: span 2;
}

@media (max-width: 900px) {
  .ap-grid,
  .ap-two,
  .ap-flowline {
    grid-template-columns: 1fr;
  }
}
</style>

---
layout: full
---

<ContextFlow />

---
layout: full
---

<InfiniteFlow />
