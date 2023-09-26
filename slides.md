---
theme: default
background: https://images.unsplash.com/photo-1502239608882-93b729c6af43?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2940&q=80
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
title: Vue Composables
mdc: true
---

# Vue Composables

Pattern for organizing and encapsulating logics in Vue

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/hazrinatn" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# Table of contents

<Toc maxDepth="1"></Toc>

---

# What is Composable Function?

<br>
<div v-click="1"><em>A function that leverages Vue's Composition API to encapsulate and reuse stateful logic.</em></div>

<br>
<div v-click="2">LEGO bricks for components.</div>

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->

---

# Composables vs Utility Functions vs Mixins

<div grid="~ cols-3 gap-6">
<div v-click="1">
<h4>Utility Functions</h4>

<p>
<span>General-Purpose:</span> not specific to component or component lifecycle.

<span>Stateless:</span> stateless and don't hold or manage any internal state.

<span>Low-Level:</span> often lower-level and can be used for a wide range of tasks, from simple calculations to more complex operations.

</p>

</div>
<div v-click="2">
<h4>Mixins (Vue 2)</h4>

<p>
<span>Object-Based:</span> JavaScript objects merged into a component's options (e.g., data, methods, computed).

<span>Global Scope:</span> Unexpected interactions and naming conflicts if not used carefully.

</p>
</div>

<div v-click="3">
<h4>Composables (Vue 3)</h4>

<p>
<span>Functions:</span> Return objects, easy to destructure and use.

<span>Scoped Logic:</span> Single Responsibility Principle, focuses on one aspect of behavior.

<span>Reactivity:</span> Use Vue's reactivity system (e.g., ref, reactive, computed) to manage state and trigger component updates when data changes.

<span>Explicit Use:</span> Explicitly imported and used within a component's setup function.

</p>

</div>
</div>

<style>
  h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
  p {
    font-size: 14px
  }

  span {
    font-weight: bold
  }
</style>

---

# Mouse Tracker Example

<div v-click="1">
<!-- ./components/Tracker.vue -->
<Tracker m="t-4" color="orange" />
</div>
<br>
<div v-click="2">If we were to implement the mouse tracking functionality using Composition API directly inside a component, it would look like this:</div>
<br>
<div v-click="3">

```js {all|2|4-5|7-10|12-13|all}
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const x = ref(0)
const y = ref(0)

function update(event) {
x.value = event.pageX
y.value = event.pageY
}

onMounted(() => window.addEventListener('mousemove', update))
onUnmounted(() => window.removeEventListener('mousemove', update))
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>

```

</div>

<style>
  h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

<div v-click="1">But what if we want reuse the same logic in multiple components? We can extract the logic into an external file, as a composable function:</div>

<div v-click="2">

```js {all|1-2|4-8|10-14|16-19|21-22|all}
// mouse.js
import { ref, onMounted, onUnmounted } from "vue";

// by convention, composable function names start with "use"
export function useMouse() {
  // state encapsulated and managed by the composable
  const x = ref(0);
  const y = ref(0);

  // a composable can update its managed state over time.
  function update(event) {
    x.value = event.pageX;
    y.value = event.pageY;
  }

  // a composable can also hook into owner component's lifecycle
  // to setup and teardown side effects.
  onMounted(() => window.addEventListener("mousemove", update));
  onUnmounted(() => window.removeEventListener("mousemove", update));

  // expose managed state as return value
  return { x, y };
}
```

</div>

---

And this is how it can be used in components:

```js {all|2|4|7|all}
<script setup>
import { useMouse } from './mouse.js'

const { x, y } = useMouse()
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>
```

<br>
<div v-click="4">
<!-- ./components/Tracker.vue -->
<Tracker m="t-4" color="orange" />
</div>
<br>
<div v-click="5">
As we can see, the core logic remains exactly the same - all we had to do was moving it into an external function and return the state that should be exposed. The same `useMouse()` functionality can now be used in any component.
</div>

---

Furthermore: We can extract the logic of adding and cleaning up a DOM event listener

```js {all|1-2|4-7|all}
// event.js
import { onMounted, onUnmounted } from "vue";

export function useEventListener(target, event, callback) {
  onMounted(() => target.addEventListener(event, callback));
  onUnmounted(() => target.removeEventListener(event, callback));
}
```

<div v-click="5">
And now our `useMouse()` can be simplified to:
</div>
<div v-click="6">

```js {all|all|3|9-12|all}
// mouse.js
import { ref } from "vue";
import { useEventListener } from "./event";

export function useMouse() {
  const x = ref(0);
  const y = ref(0);

  useEventListener(window, "mousemove", (event) => {
    x.value = event.pageX;
    y.value = event.pageY;
  });

  return { x, y };
}
```

</div>

<div v-click="4">
<!-- ./components/Tracker.vue -->
<Tracker m="t-4" color="yellow" />
</div>
<br>

---

# One Thing at a Time

Just the same as writing JavaScript Functions

- Extract duplicated logics into composable functions
- Have meaningful names
- Consistent naming conventions - `useXX`
- Keep functions small and simple
- "Do one thing and do it well"

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# NDS

Until now we still use mixins (handleInactiveUser.js & datePickerComponent.js) to extract component logic into reusable units in NDS micro UIs.

<div v-click="1">
There are two primary drawbacks for mixins:
<br>
<br>
<div v-click="2">

1. <span>Unclear source of properties: </span>it becomes unclear which instance property is injected by which mixin, making it difficult to trace the implementation and understand the component's behavior. This is also why we recommend using the refs + destructure pattern for composables: it makes the property source clear in consuming components.

</div>

<div v-click="3">

2. <span>Namespace collisions: </span> multiple mixins from different authors can potentially register the same property keys, causing namespace collisions. With composables, you can rename the destructured variables in case there are conclicting keys from different composables.

</div>

</div>

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
span {
  color: orange
}
</style>

---

# Learn More

[Vue JS Official Docs](https://staging-cf.vuejs.org/guide/reusability/composables.html) · [Anthony Fu - Composable Vue](https://demo.sli.dev/composable-vue/1) · [Vue School - What is a Vue.js Composable?](https://vueschool.io/articles/vuejs-tutorials/what-is-a-vue-js-composable/)
