### Vue 3 Reactivity Landmines (Composition API + `<script setup>`)

1. **Destructuring reactive objects / props breaks reactivity**  
   ```ts
   const state = reactive({ count: 0 })
   const { count } = state     // ← count is now plain number, no reactivity!
   ```
   - **Fix**: Use `toRefs` (Vue 3.5+ has reactive destructuring for props)  
     ```ts
     const { count } = toRefs(state)
     // or in <script setup> for props (Vue 3.5+)
     const { title } = defineProps<{ title: string }>()  // reactive by default in 3.5+
     ```
   - **React dev trap**: Feels like object destructuring in useState — but Vue proxies are lost.

2. **Adding new properties to reactive() after creation (sometimes)**  
   Vue 3 usually tracks new props automatically, but if you replace the whole object or use non-proxy-friendly patterns → reactivity dies.  
   ```ts
   let user = reactive({ name: 'John' })
   user = { name: 'Jane', age: 30 }  // ← new reference → lost reactivity
   ```
   - **Fix**: Mutate in place or use `ref` for the whole thing.

3. **Forgetting `.value` on ref() — or using it inconsistently**  
   ```ts
   const count = ref(0)
   count++               // TypeError or silent fail
   ```
   - **Fix**: Always `count.value++` inside `<script setup>`.  
     React/Angular devs often forget because there's no equivalent "unwrap" syntax.

4. **Calling composables inside if/for/while (conditional / loops)**  
   ```ts
   if (condition) {
     const data = useFetch()  // ← composables MUST be at top level!
   }
   ```
   - **Fix**: Call at top level; use early return or ternaries instead.

5. **Mutating props directly**  
   ```ts
   const props = defineProps<{ items: string[] }>()
   props.items.push('new')  // ← warning + broken parent expectation
   ```
   - **Fix**: Emit update or use v-model / defineModel().

6. **Watching arrays/objects incorrectly (deep: true missing)**  
   ```ts
   watch(items, () => {})          // won't trigger on array.push()
   ```
   - **Fix**: `watch(items, ..., { deep: true })` or watch a ref to the array.

7. **Reassigning ref variables (loses reactivity)**  
   ```ts
   let count = ref(0)
   count = ref(10)  // ← new ref, watchers on old one are lost
   ```
   - **Fix**: Mutate `.value` or use a reactive container.

8. **Not cleaning up side effects (memory leaks)**  
   ```ts
   onMounted(() => {
     window.addEventListener('resize', handler)
     // no onUnmounted removal → leak!
   })
   ```
   - **Fix**: Return cleanup from onMounted or use onUnmounted.

9. **Expecting reactivity from non-reactive sources**  
   - `localStorage.getItem()`  
   - Plain JS objects/maps/sets not wrapped  
   - Third-party libs without Vue integration  
   - **Fix**: Wrap in ref/reactive or use watch + manual trigger.

10. **Using index as :key in v-for (performance + bugs)**  
    ```vue
    <div v-for="(item, index) in list" :key="index">  <!-- bad when list reorders -->
    ```
    - **Fix**: Use unique id from data (`:key="item.id"`).

### Quick Rules of Thumb (Memorize These)

- Prefer `ref()` for most primitives & single values — easier mental model.  
- Use `reactive()` for complex state objects you mutate a lot (but avoid destructuring).  
- **Never** destructure props/state without `toRefs` (pre-3.5 especially).  
- Composables → **top level only**, no conditions/loops.  
- Props are **readonly** — mutate via emits or v-model.  
- Always ask: "Does this create a new reference?" → if yes, reactivity likely lost.
