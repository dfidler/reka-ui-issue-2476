# reak-ui Issue 2476  (TabsContent forceMount bug)

## Bug

https://github.com/unovue/reka-ui/issues/2476

In SPA (non-SSR) mode, `TabsContent` internally hardcodes `force-mount` on its
`Presence` component. This means **all tab panels remain in the DOM** even when
inactive — they are hidden only via the `hidden` HTML attribute.

The `hidden` attribute works by setting `display: none`, but any CSS that
explicitly sets `display` (e.g. `display: flex`, `display: grid`) overrides it,
causing inactive tab content to be visible alongside the active tab.

This is a common scenario — flex/grid layouts inside tabs are widespread,
especially with utility-first CSS frameworks like Tailwind (`class="flex"`).

## Reproduction

```bash
npm install
npm run dev
```

Click between tabs — all three tab panels are visible simultaneously.

## Root cause

In `TabsContent.vue`, the internal `Presence` component has `force-mount`
hardcoded (added in commit `9526356` as an SSR hydration fix):

```vue
<Presence
  v-slot="{ present }"
  :present="forceMount || isSelected"
  force-mount          <!-- always force-mounted, even in SPA -->
>
```

This keeps inactive tab panels in the DOM with `hidden` attribute instead of
removing them. Any `display` CSS override defeats `hidden`.

## Proposed fix

Only force-mount during SSR, using the existing `isBrowser` utility:

```vue
<Presence
  v-slot="{ present }"
  :present="forceMount || isSelected"
  :force-mount="!isBrowser || forceMount"
>
```

This preserves the SSR hydration fix while allowing SPA apps to properly
unmount inactive tabs.

## Workaround

Since `TabsContent` keeps inactive panels in the DOM, you need to manually
control visibility using the `TabsRoot` model value:

```vue
<script setup>
import { ref } from 'vue'
import { TabsRoot, TabsList, TabsTrigger, TabsContent } from 'reka-ui'

const activeTab = ref('one')
</script>

<template>
  <TabsRoot v-model="activeTab">
    <TabsList>
      <TabsTrigger value="one">Tab 1</TabsTrigger>
      <TabsTrigger value="two">Tab 2</TabsTrigger>
    </TabsList>

    <TabsContent value="one">
      <div v-if="activeTab === 'one'" class="flex flex-col">
        Tab 1 content
      </div>
    </TabsContent>

    <TabsContent value="two">
      <div v-if="activeTab === 'two'" class="flex flex-col">
        Tab 2 content
      </div>
    </TabsContent>
  </TabsRoot>
</template>
```
