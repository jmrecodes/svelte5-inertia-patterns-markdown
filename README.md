# Svelte 5 + Inertia.js Patterns & Best Practices

## 📚 Complete Reference Guide

This document contains all the patterns, solutions, and best practices we discovered while building a production-ready Svelte 5 + Inertia.js application. Every pattern here is battle-tested and educational.

## 🎯 Core Principles

1. **Use Official APIs**: Always prefer documented, official patterns over community workarounds
2. **Keep It Simple**: Complex solutions often indicate you're fighting the framework
3. **Consistency**: Use the same patterns across all components for maintainability
4. **Education First**: Code should be clear and teachable, not just functional

## 🔧 Essential Setup Patterns

### **1. App Initialization (app.js)**

```javascript
import { createInertiaApp } from '@inertiajs/svelte'
import { mount } from 'svelte'

createInertiaApp({
  title: (title) => `${title} - Educational Blog`,
  
  resolve: (name) => {
    const pages = import.meta.glob('./Pages/**/*.svelte', { eager: true })
    return pages[`./Pages/${name}.svelte`]
  },
  
  setup({ el, App, props }) {
    // CRITICAL: Clear element and use Svelte 5 mount() API
    el.innerHTML = ''
    const app = mount(App, {
      target: el,
      props,
    })
    return app
  },
  
  progress: {
    color: '#4F46E5',
    showSpinner: true,
  },
})
```

**Key Points:**
- ✅ Use `mount()` instead of `new App()`
- ✅ Clear element with `el.innerHTML = ''`
- ✅ Import from `@inertiajs/svelte` (not legacy packages)

### **2. Vite Configuration**

```javascript
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import { svelte } from '@sveltejs/vite-plugin-svelte'

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/css/app.css', 'resources/js/app.js'],
      refresh: true,
    }),
    
    // Simple single plugin - no complex dual setup needed
    svelte({
      compilerOptions: {
        dev: process.env.NODE_ENV === 'development',
        hmr: process.env.NODE_ENV === 'development',
      },
      emitCss: process.env.NODE_ENV === 'production',
    }),
  ],
  
  server: {
    hmr: {
      host: 'localhost',
    },
  },
})
```

**Key Points:**
- ✅ Single Svelte plugin configuration
- ✅ No complex include/exclude patterns needed
- ✅ Simple, maintainable setup

## 📝 Form Handling Patterns

### **❌ Don't Use `useForm` with Svelte 5**

```javascript
// THIS CAUSES REACTIVITY ISSUES
import { useForm } from '@inertiajs/svelte'
let form = useForm({ email: '', password: '' })
```

### **✅ Use Official router.post() Pattern**

```javascript
import { router } from '@inertiajs/svelte'

// 1. Props (include errors from server)
let { errors = {} } = $props()

// 2. Reactive form values
let values = $state({
  email: '',
  password: '',
  remember: false
})

// 3. Manual processing state
let processing = $state(false)

// 4. Form submission
function handleSubmit(event) {
  event.preventDefault()
  processing = true
  
  router.post('/login', values, {
    onSuccess: () => {
      console.log('Success!')
      processing = false
    },
    onError: (errors) => {
      console.log('Errors:', errors)
      processing = false  
    },
    onFinish: () => {
      processing = false
    }
  })
}
```

**Template Usage:**
```svelte
<form onsubmit={handleSubmit}>
  <input 
    bind:value={values.email}
    disabled={processing}
    class={errors?.email ? 'border-red-500' : ''}
  />
  
  {#if errors?.email}
    <p class="text-red-600">{errors.email}</p>
  {/if}
  
  <button type="submit" disabled={processing}>
    {processing ? 'Loading...' : 'Submit'}
  </button>
</form>
```

## 🧮 Svelte 5 Runes Best Practices

### **$state() - Reactive Local State**

```javascript
// ✅ Simple reactive values
let isLoading = $state(false)
let showMenu = $state(false)

// ✅ Objects and arrays  
let user = $state({ name: '', email: '' })
let items = $state([])
```

### **$props() - Component Props**

```javascript
// ✅ With defaults and destructuring
let { 
  user = null,
  errors = {},
  flashMessage = null 
} = $props()

// ✅ Access server data from Laravel/Inertia
let { posts, pagination, filters } = $props()
```

### **$derived() - Computed Values**

```javascript
// ✅ Simple expressions work best
let isAuthenticated = $derived(user !== null)
let hasErrors = $derived(Object.keys(errors).length > 0)

// ✅ Ternary operators for complex logic
let greeting = $derived(
  currentTime.getHours() < 12 ? 'Good morning' :
  currentTime.getHours() < 17 ? 'Good afternoon' : 
  'Good evening'
)

// ✅ Function calls with simple expressions
let formattedDate = $derived(formatDate(createdAt))

// ❌ Avoid arrow functions - they cause rendering issues
let computed = $derived(() => {
  // This can cause display problems
  return complexCalculation()
})
```

### **$effect() - Side Effects**

```javascript
// ✅ Simple side effects
$effect(() => {
  document.title = `Dashboard - ${user?.name || 'User'}`
})

// ✅ Cleanup functions
$effect(() => {
  const interval = setInterval(() => {
    currentTime = new Date()
  }, 60000)
  
  return () => {
    clearInterval(interval)
  }
})
```

## 🎨 Event Handling (Svelte 5 Syntax)

```svelte
<!-- ✅ Click handlers -->
<button onclick={() => showMenu = !showMenu}>
  Toggle Menu
</button>

<!-- ✅ Form submission -->
<form onsubmit={handleSubmit}>
  <!-- form content -->
</form>

<!-- ✅ Input events -->
<input 
  oninput={(e) => values.email = e.target.value}
  onfocus={() => focusedField = 'email'}
  onblur={() => focusedField = null}
/>

<!-- ✅ Conditional event handlers -->
<button 
  onclick={processing ? null : handleSubmit}
  disabled={processing}
>
  Submit
</button>
```

## 🏗️ Component Structure Pattern

Every component should follow this consistent structure:

```javascript
<script>
  // 1. IMPORTS
  import { router, Link } from '@inertiajs/svelte'
  import { formatDate } from '../Utils/helpers.js'
  
  // 2. PROPS (server data)
  let { 
    user = null,
    errors = {},
    posts = []
  } = $props()
  
  // 3. LOCAL STATE
  let isLoading = $state(false)
  let selectedItem = $state(null)
  
  // 4. COMPUTED VALUES
  let isAuthenticated = $derived(user !== null)
  let postCount = $derived(posts.length)
  
  // 5. EVENT HANDLERS
  function handleAction() {
    // Implementation
  }
  
  // 6. EFFECTS (if needed)
  $effect(() => {
    // Side effects
  })
</script>

<!-- 7. TEMPLATE -->
<div>
  <!-- Component content -->
</div>
```

## 🔐 Authentication Patterns

### **Login Component**
```javascript
// Simple, consistent pattern
let values = $state({ email: '', password: '', remember: false })
let processing = $state(false)

function handleLogin(event) {
  event.preventDefault()
  processing = true
  
  router.post('/login', values, {
    onSuccess: () => processing = false,
    onError: () => processing = false,
    onFinish: () => processing = false
  })
}
```

### **Dashboard Access**
```javascript
// Props from authenticated route
let { user, stats, recentActivity } = $props()

// Computed greeting
let greeting = $derived(
  new Date().getHours() < 12 ? 'Good morning' :
  new Date().getHours() < 17 ? 'Good afternoon' : 
  'Good evening'
)
```

### **Logout Handler**
```javascript
async function handleLogout() {
  if (isLoggingOut) return
  isLoggingOut = true
  
  router.post('/logout', {}, {
    onSuccess: () => console.log('Logged out'),
    onError: () => isLoggingOut = false,
  })
}
```

## 📋 Form Validation Patterns

### **Real-time Validation**
```javascript
let validation = $derived({
  email: {
    isValid: /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email),
    message: values.email.length === 0 ? '' : 
             'Please enter a valid email'
  },
  password: {
    isValid: values.password.length >= 8,
    message: values.password.length === 0 ? '' :
             'Password must be at least 8 characters'
  }
})

let isFormValid = $derived(
  validation.email.isValid && 
  validation.password.isValid
)
```

### **Error Display**
```svelte
<!-- Server errors from Laravel -->
{#if errors?.email}
  <p class="text-red-600">{errors.email}</p>
{/if}

<!-- Client validation -->
{#if validation.email.message}
  <p class="text-red-600">{validation.email.message}</p>
{/if}

<!-- Visual indicators -->
<input 
  class={errors?.email ? 'border-red-500' : 
         validation.email.isValid ? 'border-green-500' : ''}
/>
```

## 🎯 Common Patterns Summary

### **Do Use ✅**
- `router.post()` for form submissions
- `$state()` for reactive values
- Simple expressions in `$derived()`
- `onclick={handler}` for events
- Props destructuring with defaults
- Manual processing state management

### **Don't Use ❌**
- `useForm()` with Svelte 5 (causes issues)
- Arrow functions in `$derived()`
- Complex logic in computed values
- `new Component()` instantiation
- Nested reactive statements

## 🎓 Educational Value

This pattern collection demonstrates:

1. **Modern Web Development**: Latest Svelte 5 and Inertia.js patterns
2. **Problem Solving**: How we discovered and solved integration issues
3. **Best Practices**: Consistent, maintainable code patterns
4. **Real-world Application**: Patterns used in production applications
5. **Progressive Enhancement**: Forms work without JavaScript

## 📚 References

- [Inertia.js Forms Documentation](https://inertiajs.com/forms)
- [Svelte 5 Documentation](https://svelte.dev/docs)
- [Laravel Inertia Documentation](https://inertiajs.com/)

---

*This guide represents our complete learning journey from broken implementation to production-ready patterns. Every recommendation is based on real experience and extensive testing.* 