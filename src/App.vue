<script setup>
import { TabsRoot, TabsList, TabsTrigger, TabsContent } from 'reka-ui'
</script>

<template>
  <div style="padding: 16px;">
    <h3>reka-ui Tabs: force-mount keeps inactive tabs in DOM</h3>
    <p style="font-size: 12px; color: #666; margin-bottom: 8px;">
      Inactive TabsContent elements are force-mounted and hidden via the
      <code>hidden</code> attribute. Any CSS that sets <code>display</code>
      (e.g. <code>display: flex</code>) overrides <code>hidden</code>,
      making inactive tabs visible.
    </p>

    <TabsRoot default-value="one" style="display: flex; flex-direction: column;">
      <TabsList style="display: flex; gap: 4px; margin-bottom: 8px;">
        <TabsTrigger value="one" style="padding: 8px 16px; cursor: pointer;">Tab 1</TabsTrigger>
        <TabsTrigger value="two" style="padding: 8px 16px; cursor: pointer;">Tab 2</TabsTrigger>
        <TabsTrigger value="three" style="padding: 8px 16px; cursor: pointer;">Tab 3</TabsTrigger>
      </TabsList>

      <TabsContent value="one" style="padding: 16px; background: #d32f2f; color: white;">
        <p>Tab 1 content</p>
      </TabsContent>

      <TabsContent value="two" style="padding: 16px; background: #2e7d32; color: white;">
        <p>Tab 2 content</p>
      </TabsContent>

      <TabsContent value="three" style="padding: 16px; background: #1565c0; color: white;">
        <p>Tab 3 content</p>
      </TabsContent>
    </TabsRoot>
  </div>
</template>

<style>
/* This demonstrates the bug: any display override defeats hidden */
[role="tabpanel"][hidden] {
  display: block !important;
}
</style>
