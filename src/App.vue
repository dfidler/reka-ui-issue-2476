<script setup>
import { nextTick, onMounted, ref, watch } from 'vue'
import { TabsContent, TabsList, TabsRoot, TabsTrigger } from 'reka-ui'
import { version as rekaVersion } from 'reka-ui/package.json'

const tabs = [
  { value: 'one', label: 'Tab 1' },
  { value: 'two', label: 'Tab 2' },
  { value: 'three', label: 'Tab 3' },
]
const rows = Array.from({ length: 20 }, (_, i) => i + 1)

const bugTab = ref('one')
const fixTab = ref('one')
const bugFrame = ref(null)
const fixFrame = ref(null)
const bugStats = ref({ inDom: 0, visible: 0, panels: [] })
const fixStats = ref({ inDom: 0, visible: 0, panels: [] })

function measure(frame) {
  const panels = [...frame.querySelectorAll('[role="tabpanel"]')]
  return {
    inDom: panels.length,
    visible: panels.filter((panel) => panel.getClientRects().length > 0).length,
    panels: panels.map((panel) => ({
      id: panel.id,
      state: panel.dataset.state,
      // Opening tag only, exactly as the browser holds it.
      tag: panel.cloneNode(false).outerHTML.replace(/<\/div>$/, ''),
      inner: panel.childElementCount === 0 ? panel.innerHTML.trim() || '(empty)' : `${panel.childElementCount} child elements`,
      display: getComputedStyle(panel).display,
      height: Math.round(panel.getBoundingClientRect().height),
    })),
  }
}

async function refresh() {
  await nextTick()
  bugStats.value = measure(bugFrame.value)
  fixStats.value = measure(fixFrame.value)
}

onMounted(refresh)
watch([bugTab, fixTab], refresh)
</script>

<template>
  <main class="page">
    <h1>reka-ui #2476 &mdash; inactive <code>TabsContent</code> panels consume layout space</h1>
    <p class="intro">
      Every <code>TabsContent</code> stays in the DOM as an empty
      <code>&lt;div role="tabpanel" hidden&gt;</code>. The <code>hidden</code> attribute only
      works through the browser's default <code>display: none</code>, so any author rule that
      sets <code>display</code> on the panel (here <code>display: flex</code>, i.e. Tailwind's
      <code>class="flex"</code>) makes the empty panels visible. In a flex column they each
      claim an equal share of the height, and the active panel is squeezed and scrolls.
    </p>

    <div class="columns">
      <section ref="bugFrame" class="frame">
        <h2>Current code (reka-ui {{ rekaVersion }})</h2>
        <TabsRoot v-model="bugTab" class="tabs">
          <TabsList class="tabs-list">
            <TabsTrigger v-for="tab in tabs" :key="tab.value" :value="tab.value" class="tabs-trigger">
              {{ tab.label }}
            </TabsTrigger>
          </TabsList>
          <TabsContent v-for="tab in tabs" :key="tab.value" :value="tab.value" class="panel" :class="`panel-${tab.value}`">
            <div v-for="row in rows" :key="row" class="row">{{ tab.label }} &middot; row {{ row }}</div>
          </TabsContent>
        </TabsRoot>
        <p class="stats bad">
          {{ bugStats.inDom }} &times; <code>[role=tabpanel]</code> in DOM &middot;
          <strong>{{ bugStats.visible }} visible</strong>
        </p>
      </section>
      <div class="dom">
        <div class="dom-title">Live DOM &amp; computed style of each <code>[role=tabpanel]</code></div>
        <div v-for="panel in bugStats.panels" :key="panel.id" class="dom-panel" :class="panel.state">
          <code class="dom-tag">{{ panel.tag }}</code>
          <code class="dom-tag dom-inner">&nbsp;&nbsp;{{ panel.inner }}</code>
          <div class="dom-computed">
            <span>display: <b :class="panel.display === 'none' ? 'ok' : 'bad'">{{ panel.display }}</b></span>
            <span>height: <b :class="panel.display === 'none' ? 'ok' : 'bad'">{{ panel.height }}px</b></span>
          </div>
        </div>
      </div>

      <section ref="fixFrame" class="frame with-fix">
        <h2>Expected (with <code>[data-state=inactive] { display: none }</code>)</h2>
        <TabsRoot v-model="fixTab" class="tabs">
          <TabsList class="tabs-list">
            <TabsTrigger v-for="tab in tabs" :key="tab.value" :value="tab.value" class="tabs-trigger">
              {{ tab.label }}
            </TabsTrigger>
          </TabsList>
          <TabsContent v-for="tab in tabs" :key="tab.value" :value="tab.value" class="panel" :class="`panel-${tab.value}`">
            <div v-for="row in rows" :key="row" class="row">{{ tab.label }} &middot; row {{ row }}</div>
          </TabsContent>
        </TabsRoot>
        <p class="stats good">
          {{ fixStats.inDom }} &times; <code>[role=tabpanel]</code> in DOM &middot;
          <strong>{{ fixStats.visible }} visible</strong>
        </p>
      </section>
      <div class="dom">
        <div class="dom-title">Live DOM &amp; computed style of each <code>[role=tabpanel]</code></div>
        <div v-for="panel in fixStats.panels" :key="panel.id" class="dom-panel" :class="panel.state">
          <code class="dom-tag">{{ panel.tag }}</code>
          <code class="dom-tag dom-inner">&nbsp;&nbsp;{{ panel.inner }}</code>
          <div class="dom-computed">
            <span>display: <b :class="panel.display === 'none' ? 'ok' : 'bad'">{{ panel.display }}</b></span>
            <span>height: <b :class="panel.display === 'none' ? 'ok' : 'bad'">{{ panel.height }}px</b></span>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>


<style>
.page {
  max-width: 1160px;
  margin: 0 auto;
  padding: 24px;
}
h1 {
  font-size: 20px;
  margin: 0 0 8px;
}
h2 {
  font-size: 14px;
  margin: 0 0 8px;
  color: #444;
}
.intro {
  font-size: 13px;
  color: #555;
  margin: 0 0 20px;
  max-width: 900px;
}
code {
  font-size: 0.92em;
  background: #f1f3f5;
  padding: 1px 4px;
  border-radius: 3px;
}
.columns {
  display: grid;
  grid-template-columns: 1fr 1fr;
  grid-auto-flow: dense;
  gap: 16px 24px;
}
.columns > .frame:nth-of-type(1) { grid-column: 1; grid-row: 1; }
.columns > .frame:nth-of-type(2) { grid-column: 2; grid-row: 1; }
.columns > .dom:nth-of-type(1) { grid-column: 1; grid-row: 2; }
.columns > .dom:nth-of-type(2) { grid-column: 2; grid-row: 2; }

/* DevTools-style "Elements + Computed" pane */
.dom {
  border: 1px solid #ddd;
  border-radius: 8px;
  background: #fff;
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
  font-size: 11.5px;
  overflow: hidden;
}
.dom-title {
  font-family: system-ui, sans-serif;
  font-size: 12px;
  color: #444;
  padding: 6px 10px;
  background: #f3f3f3;
  border-bottom: 1px solid #ddd;
}
.dom-title code { background: #e4e4e4; }
.dom-panel {
  padding: 6px 10px;
  border-bottom: 1px solid #eee;
}
.dom-panel:last-child { border-bottom: 0; }
.dom-panel.inactive { background: #fff3e0; }
.dom-tag {
  display: block;
  white-space: pre-wrap;
  word-break: break-all;
  background: none;
  padding: 0;
  color: #881280;
}
.dom-inner { color: #236e25; }
.dom-computed {
  display: flex;
  gap: 18px;
  margin-top: 4px;
  color: #555;
}
.dom-computed b { color: #222; }
.dom-panel.inactive .dom-computed b.bad { color: #b71c1c; }
.dom-panel.inactive .dom-computed b.ok { color: #2e7d32; }

/* A fixed-height flex column: the classic app shell for a tabbed panel. */
.frame {
  height: 440px;
  display: flex;
  flex-direction: column;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 12px;
  background: #fafafa;
}
.tabs {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
}
.tabs-list {
  display: flex;
  gap: 4px;
  margin-bottom: 8px;
}
.tabs-trigger {
  padding: 6px 14px;
  border: 1px solid #bbb;
  border-radius: 6px 6px 0 0;
  background: #eee;
  font: inherit;
  cursor: pointer;
}
.tabs-trigger[data-state='active'] {
  background: #fff;
  border-bottom-color: #fff;
  font-weight: 600;
}

/*
 * The only thing that matters: the panel sets `display`. Tailwind's `flex flex-col flex-1
 * min-h-0 overflow-y-auto` produces exactly these declarations. No !important anywhere.
 */
.panel {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  position: relative;
  border: 2px solid;
  border-radius: 6px;
  background: #fff;
}
.panel-one { border-color: #d32f2f; }
.panel-two { border-color: #2e7d32; }
.panel-three { border-color: #1565c0; }

.row {
  padding: 4px 10px;
  font-size: 13px;
  font-variant-numeric: tabular-nums;
}
.row:nth-child(odd) { background: #e3f2fd; }
.row:nth-child(even) { background: #ffffff; }

/* Label every panel so the screenshot explains itself. */
.panel::before {
  position: sticky;
  top: 0;
  display: block;
  padding: 3px 10px;
  font-size: 11px;
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
  color: #fff;
  background: #333;
}
.panel[data-state='active']::before {
  content: attr(id) '  data-state=active';
}
.panel[data-state='inactive']::before {
  content: attr(id) '  data-state=inactive  hidden=""  (empty, but display:flex beats hidden)';
  background: #b71c1c;
}
.panel[data-state='inactive'] {
  background: repeating-linear-gradient(135deg, #fff, #fff 8px, #fde8e8 8px, #fde8e8 16px);
}

/* The workaround the maintainers suggest (Tailwind: data-[state=inactive]:hidden). */
.with-fix .panel[data-state='inactive'] {
  display: none;
}

.stats {
  margin: 10px 0 0;
  font-size: 13px;
}
.stats.bad strong { color: #b71c1c; }
.stats.good strong { color: #2e7d32; }
</style>
