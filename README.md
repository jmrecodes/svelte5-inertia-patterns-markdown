---
description: cursor rule for implementation of components in Svelte 5 with Inertia.js
globs: .svelte
alwaysApply: false
---
# Svelte 5 + Inertia.js Complete Implementation Guide

## 📚 Production-Ready Patterns & Best Practices

This document contains all the patterns, solutions, and best practices from building a comprehensive full-stack blog application with **Svelte 5**, **Inertia.js 2.0**, **Laravel 12**, and **Tailwind CSS v4**. Every pattern here is battle-tested, production-ready, and educational.

## 🎯 Core Principles

1. **Use Official APIs**: Always prefer documented, official patterns over community workarounds
2. **Keep It Simple**: Complex solutions often indicate you're fighting the framework
3. **Consistency**: Use the same patterns across all components for maintainability
4. **Education First**: Code should be clear and teachable, not just functional
5. **Production Ready**: All patterns are tested in a real application with authentication, CRUD operations, and advanced features

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

### **2. Vite Configuration (Complete)**

```javascript
import { defineConfig } from 'vite'
import laravel from 'laravel-vite-plugin'
import { svelte } from '@sveltejs/vite-plugin-svelte'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/css/app.css', 'resources/js/app.js'],
      refresh: true,
    }),
    
    // Tailwind CSS v4 direct integration
    tailwindcss(),
    
    // Svelte 5 with runes support
    svelte({
      compilerOptions: {
        dev: process.env.NODE_ENV === 'development',
        hmr: process.env.NODE_ENV === 'development',
      },
      emitCss: process.env.NODE_ENV === 'production',
    }),
  ],
  
  server: {
    host: true,
    hmr: {
      host: 'localhost',
    },
  },
  
  build: {
    sourcemap: process.env.NODE_ENV === 'development',
    rollupOptions: {
      output: {
        manualChunks: {
          svelte: ['svelte'],
          inertia: ['@inertiajs/svelte'],
        },
      },
    },
  },
  
  resolve: {
    alias: {
      '@': '/resources/js',
      '@components': '/resources/js/Components',
      '@pages': '/resources/js/Pages',
    },
  },
})
```

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

### **✅ Advanced Form Pattern with Validation**

```javascript
// Complete form pattern with client-side validation
let values = $state({
  title: '',
  content: '',
  meta_description: '',
  status: 'draft'
})

let processing = $state(false)
let { errors = {} } = $props()

// Real-time validation
let validation = $derived({
  title: {
    isValid: values.title.trim().length >= 3,
    message: values.title.length === 0 ? '' : 
             values.title.trim().length < 3 ? 'Title must be at least 3 characters' : ''
  },
  content: {
    isValid: values.content.trim().length >= 10,
    message: values.content.length === 0 ? '' :
             values.content.trim().length < 10 ? 'Content must be at least 10 characters' : ''
  }
})

let isFormValid = $derived(
  validation.title.isValid && 
  validation.content.isValid &&
  !processing
)

function handleSubmit(event) {
  event.preventDefault()
  if (!isFormValid) return
  
  processing = true
  
  router.post('/posts', values, {
    onSuccess: () => {
      // Reset form on success
      values = { title: '', content: '', meta_description: '', status: 'draft' }
    },
    onError: (errors) => {
      console.log('Server validation errors:', errors)
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
    bind:value={values.title}
    disabled={processing}
    placeholder="Post Title"
    class={`border rounded px-3 py-2 ${
      errors?.title || validation.title.message ? 'border-red-500' : 
      validation.title.isValid && values.title.length > 0 ? 'border-green-500' : 'border-gray-300'
    }`}
  />
  
  {#if errors?.title}
    <p class="text-red-600 text-sm mt-1">{errors.title}</p>
  {:else if validation.title.message}
    <p class="text-red-600 text-sm mt-1">{validation.title.message}</p>
  {/if}
  
  <textarea 
    bind:value={values.content}
    disabled={processing}
    placeholder="Write your post content..."
    rows="10"
    class={`border rounded px-3 py-2 w-full ${
      errors?.content || validation.content.message ? 'border-red-500' : 
      validation.content.isValid && values.content.length > 0 ? 'border-green-500' : 'border-gray-300'
    }`}
  ></textarea>
  
  <button 
    type="submit" 
    disabled={!isFormValid}
    class={`px-4 py-2 rounded ${
      isFormValid ? 'bg-blue-600 hover:bg-blue-700 text-white' : 
      'bg-gray-300 text-gray-500 cursor-not-allowed'
    }`}
  >
    {processing ? 'Creating...' : 'Create Post'}
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

// ✅ Complex form state
let blogPost = $state({
  title: '',
  content: '',
  slug: '',
  status: 'draft',
  meta_description: '',
  featured_image: null
})

// ✅ Search and filters state
let filters = $state({
  search: '',
  status: 'all',
  sort: 'created_at',
  direction: 'desc'
})

// 💡 LEARN: All values managed by $state are deeply reactive.
// This means if you change a property of an object or an item in an array,
// Svelte will automatically detect the change and update the UI.
```

### **Two-Way Binding (`bind:value`) - Effortless Form Sync**

```svelte
<!-- ✅ Text inputs with real-time updates -->
<input 
  type="email" 
  bind:value={values.email} 
  placeholder="Your Email"
  class="border rounded px-3 py-2"
/>

<!-- ✅ Textarea for content -->
<textarea 
  bind:value={values.content} 
  placeholder="Write your post..."
  rows="8"
  class="w-full border rounded px-3 py-2"
></textarea>

<!-- ✅ Select dropdowns -->
<select bind:value={values.status} class="border rounded px-3 py-2">
  <option value="draft">Draft</option>
  <option value="published">Published</option>
  <option value="private">Private</option>
</select>

<!-- ✅ Checkboxes for boolean values -->
<label class="flex items-center">
  <input 
    type="checkbox" 
    bind:checked={values.featured}
    class="mr-2"
  />
  Featured Post
</label>

<!-- ✅ Number inputs with automatic type conversion -->
<input 
  type="number" 
  bind:value={values.sort_order}
  min="1"
  class="border rounded px-3 py-2"
/>

<!-- ✅ Search input with debounced updates -->
<input 
  type="search" 
  bind:value={searchQuery}
  placeholder="Search posts..."
  class="border rounded px-3 py-2 w-full"
/>
```

### **$props() - Component Props**

```javascript
// ✅ Authentication props
let { 
  user = null,
  errors = {},
  flashMessage = null 
} = $props()

// ✅ Blog listing props
let { 
  posts = [],
  pagination = {},
  filters = {},
  canCreatePost = false
} = $props()

// ✅ Single post props
let { 
  post,
  author,
  canEdit = false,
  canDelete = false,
  relatedPosts = []
} = $props()

// ✅ Profile management props
let { 
  profile,
  statistics = {},
  recentActivity = [],
  securitySettings = {}
} = $props()

// 💡 LEARN: Props are read-only from the parent component.
// They react to changes in the parent, but you don't modify them directly.
// Use local $state for any modifications needed within the component.
```

### **$derived() - Computed Values**

```javascript
// ✅ Authentication state
let isAuthenticated = $derived(user !== null)
let isAdmin = $derived(user?.role === 'admin')

// ✅ Form validation state
let hasErrors = $derived(Object.keys(errors).length > 0)
let canSubmit = $derived(
  values.title.trim().length >= 3 && 
  values.content.trim().length >= 10 && 
  !processing
)

// ✅ Post status and display
let postStatusLabel = $derived(
  post?.status === 'published' ? 'Published' :
  post?.status === 'draft' ? 'Draft' :
  post?.status === 'private' ? 'Private' : 'Unknown'
)

// ✅ Time-based greetings
let greeting = $derived(
  new Date().getHours() < 12 ? 'Good morning' :
  new Date().getHours() < 17 ? 'Good afternoon' : 
  'Good evening'
)

// ✅ Search and filter results
let filteredPosts = $derived(
  posts.filter(post => 
    searchQuery === '' || 
    post.title.toLowerCase().includes(searchQuery.toLowerCase())
  )
)

// ✅ Pagination calculations
let totalPages = $derived(Math.ceil(pagination.total / pagination.per_page))
let currentPage = $derived(pagination.current_page)
let hasNextPage = $derived(currentPage < totalPages)
let hasPrevPage = $derived(currentPage > 1)

// ✅ URL slug generation
let generatedSlug = $derived(
  values.title
    .toLowerCase()
    .replace(/[^a-z0-9\s-]/g, '')
    .replace(/\s+/g, '-')
    .replace(/-+/g, '-')
    .trim('-')
)
```

### **$effect() - Side Effects**

```javascript
// ✅ Document title updates
$effect(() => {
  document.title = post ? `${post.title} - Educational Blog` : 'Educational Blog'
})

// ✅ Auto-save functionality
let autoSaveTimer = $state(null)

$effect(() => {
  // Clear previous timer
  if (autoSaveTimer) {
    clearTimeout(autoSaveTimer)
  }
  
  // Set new timer if form has content
  if (values.title || values.content) {
    autoSaveTimer = setTimeout(() => {
      saveAsDraft()
    }, 30000) // Auto-save after 30 seconds of inactivity
  }
  
  return () => {
    if (autoSaveTimer) {
      clearTimeout(autoSaveTimer)
    }
  }
})

// ✅ Search debouncing
$effect(() => {
  const timer = setTimeout(() => {
    if (searchQuery !== lastSearchQuery) {
      performSearch(searchQuery)
      lastSearchQuery = searchQuery
    }
  }, 500)
  
  return () => clearTimeout(timer)
})

// ✅ Focus management
$effect(() => {
  if (showModal && modalInputRef) {
    modalInputRef.focus()
  }
})

// ✅ Scroll position restoration
$effect(() => {
  if (typeof window !== 'undefined') {
    const savedPosition = sessionStorage.getItem('scrollPosition')
    if (savedPosition) {
      window.scrollTo(0, parseInt(savedPosition))
      sessionStorage.removeItem('scrollPosition')
    }
  }
})
```

## 🎨 Event Handling (Svelte 5 Syntax)

```svelte
<!-- ✅ Click handlers -->
<button onclick={() => showMenu = !showMenu}>
  Toggle Menu
</button>

<!-- ✅ Form submission with validation -->
<form onsubmit={handleSubmit}>
  <!-- form content -->
</form>

<!-- ✅ Input events with debouncing -->
<input 
  oninput={(e) => {
    clearTimeout(searchTimer)
    searchTimer = setTimeout(() => {
      searchQuery = e.target.value
    }, 300)
  }}
  placeholder="Search..."
/>

<!-- ✅ Conditional event handlers -->
<button 
  onclick={processing ? null : handleSubmit}
  disabled={processing}
  class={processing ? 'opacity-50 cursor-not-allowed' : 'hover:bg-blue-700'}
>
  {processing ? 'Saving...' : 'Save'}
</button>

<!-- ✅ Keyboard event handling -->
<input 
  onkeydown={(e) => {
    if (e.key === 'Enter' && e.ctrlKey) {
      handleSubmit(e)
    }
    if (e.key === 'Escape') {
      closeModal()
    }
  }}
/>

<!-- ✅ File upload handling -->
<input 
  type="file"
  accept="image/*"
  onchange={(e) => {
    const file = e.target.files[0]
    if (file) {
      handleImageUpload(file)
    }
  }}
/>

<!-- ✅ Drag and drop -->
<div 
  ondrop={(e) => {
    e.preventDefault()
    const files = Array.from(e.dataTransfer.files)
    handleFilesDrop(files)
  }}
  ondragover={(e) => e.preventDefault()}
  ondragenter={() => isDragOver = true}
  ondragleave={() => isDragOver = false}
  class={`border-2 border-dashed p-8 ${isDragOver ? 'border-blue-500 bg-blue-50' : 'border-gray-300'}`}
>
  Drop files here or click to upload
</div>
```

## 🏗️ Component Structure Pattern

Every component should follow this consistent structure for readability and maintainability:

```svelte
<script>
  // 1. IMPORTS: External modules and components needed.
  import { router } from '@inertiajs/svelte' // Inertia.js router for client-side navigation   import { Link } from '@inertiajs/svelte'     // Inertia.js Link component for SPA-like navigation
  import { formatDate, generateSlug } from '../Utils/helpers.js'
  import Footer from '../Components/Footer.svelte'
  
  // 2. PROPS: Data passed from the parent component or Laravel (server-side).
  let { 
    user = null,           // Authenticated user object
    errors = {},           // Validation errors from server
    posts = [],            // List of posts for blog listing
    post = null,           // Single post object for show/edit pages
    flashMessage = null,   // Success/info messages
    canCreatePost = false, // Authorization flags
    pagination = {}        // Pagination metadata
  } = $props()
  
  // 3. LOCAL STATE: Reactive variables for internal component management.
  let isLoading = $state(false)           // Loading states
  let showConfirmDialog = $state(false)   // Modal/dialog states
  let searchQuery = $state('')            // Search functionality
  let selectedItems = $state([])          // Bulk operations
  let formValues = $state({               // Form data
    title: '',
    content: '',
    status: 'draft'
  })
  
  // 4. COMPUTED VALUES: Values derived from other reactive state or props.
  let isAuthenticated = $derived(user !== null)
  let filteredPosts = $derived(
    posts.filter(post => 
      searchQuery === '' || 
      post.title.toLowerCase().includes(searchQuery.toLowerCase())
    )
  )
  let hasUnsavedChanges = $derived(
    formValues.title !== '' || formValues.content !== ''
  )
  
  // 5. EVENT HANDLERS: Functions that respond to user interactions.
  function handleCreate(event) {
    event.preventDefault()
    if (isLoading) return
    
    isLoading = true
    router.post('/posts', formValues, {
      onSuccess: () => {
        formValues = { title: '', content: '', status: 'draft' }
      },
      onError: (errors) => {
        console.log('Validation errors:', errors)
      },
      onFinish: () => {
        isLoading = false
      }
    })
  }
  
  function handleDelete(postId) {
    if (!confirm('Are you sure you want to delete this post?')) return
    
    router.delete(`/posts/${postId}`, {
      onSuccess: () => {
        console.log('Post deleted successfully')
      }
    })
  }
  
  function handleSearch(query) {
    router.get('/posts', { search: query }, {
      preserveState: true,
      preserveScroll: true
    })
  }
  
  // 6. EFFECTS: Side effects that run when dependencies change.
  $effect(() => {
    document.title = post ? `${post.title} - Blog` : 'Blog Posts'
  })
  
  $effect(() => {
    // Auto-save functionality
    if (hasUnsavedChanges) {
      const timer = setTimeout(() => {
        saveAsDraft()
      }, 30000)
      
      return () => clearTimeout(timer)
    }
  })
</script>

<!-- 7. DYNAMIC HEAD CONTENT: SEO and metadata -->
<svelte:head>
  {#if post}
    <title>{post.title} - Educational Blog</title>
    <meta name="description" content={post.meta_description || post.excerpt || post.title} />
    
    <!-- Open Graph -->
    <meta property="og:title" content={post.title} />
    <meta property="og:description" content={post.meta_description || post.excerpt} />
    <meta property="og:type" content="article" />
    <meta property="og:url" content="{window.location.origin}/posts/{post.slug}" />
    {#if post.featured_image}
      <meta property="og:image" content={post.featured_image} />
    {/if}
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content={post.title} />
    <meta name="twitter:description" content={post.meta_description || post.excerpt} />
    
    <!-- Article specific -->
    <meta property="article:published_time" content={post.created_at} />
    <meta property="article:modified_time" content={post.updated_at} />
    <meta property="article:author" content={post.author?.name} />
    
    <!-- Canonical URL -->
    <link rel="canonical" href="{window.location.origin}/posts/{post.slug}" />
  {:else}
    <title>Blog Posts - Educational Blog</title>
    <meta name="description" content="Explore our collection of educational blog posts covering web development, programming, and technology." />
  {/if}
</svelte:head>

<!-- 8. TEMPLATE MARKUP: The visual structure -->
<div class="container mx-auto px-4 py-8">
  <!-- Header section -->
  <header class="flex justify-between items-center mb-8">
    <h1 class="text-3xl font-bold text-gray-900">
      {post ? post.title : 'Blog Posts'}
    </h1>
    
    {#if canCreatePost}
      <Link 
        href="/posts/create" 
        class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded"
      >
        Create Post
      </Link>
    {/if}
  </header>
  
  <!-- Search functionality -->
  {#if !post}
    <div class="mb-6">
      <input 
        type="search"
        bind:value={searchQuery}
        placeholder="Search posts..."
        class="w-full md:w-1/3 border border-gray-300 rounded px-3 py-2"
      />
    </div>
  {/if}
  
  <!-- Content area -->
  <main>
    {#if post}
      <!-- Single post view -->
      <article class="prose lg:prose-lg max-w-none">
        <div class="mb-4 text-sm text-gray-600">
          Published on {formatDate(post.created_at)}
          {#if post.author}
            by {post.author.name}
          {/if}
        </div>
        
        <div class="content">
          {@html post.content}
        </div>
        
        {#if user && (user.id === post.author_id || user.role === 'admin')}
          <div class="mt-8 flex gap-4">
            <Link 
              href="/posts/{post.id}/edit"
              class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded"
            >
              Edit
            </Link>
            <button 
              onclick={() => handleDelete(post.id)}
              class="bg-red-600 hover:bg-red-700 text-white px-4 py-2 rounded"
            >
              Delete
            </button>
          </div>
        {/if}
      </article>
    {:else}
      <!-- Posts listing -->
      <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {#each filteredPosts as post}
          <article class="bg-white border border-gray-200 rounded-lg p-6 hover:shadow-lg transition-shadow">
            <h2 class="text-xl font-semibold mb-2">
              <Link href="/posts/{post.id}" class="text-gray-900 hover:text-blue-600">
                {post.title}
              </Link>
            </h2>
            
            <p class="text-gray-600 mb-4">
              {post.meta_description || post.title}
            </p>
            
            <div class="flex justify-between items-center text-sm text-gray-500">
              <span>{formatDate(post.created_at)}</span>
              <span class="bg-gray-100 px-2 py-1 rounded">
                {post.status}
              </span>
            </div>
          </article>
        {/each}
      </div>
      
      <!-- Pagination -->
      {#if pagination.last_page > 1}
        <div class="mt-8 flex justify-center">
          <nav class="flex gap-2">
            {#if pagination.current_page > 1}
              <Link 
                href="/posts?page={pagination.current_page - 1}"
                class="px-3 py-2 border border-gray-300 rounded hover:bg-gray-50"
              >
                Previous
              </Link>
            {/if}
            
            {#each Array(pagination.last_page) as _, i}
              <Link 
                href="/posts?page={i + 1}"
                class={`px-3 py-2 border rounded ${
                  pagination.current_page === i + 1 
                    ? 'bg-blue-600 text-white border-blue-600' 
                    : 'border-gray-300 hover:bg-gray-50'
                }`}
              >
                {i + 1}
              </Link>
            {/each}
            
            {#if pagination.current_page < pagination.last_page}
              <Link 
                href="/posts?page={pagination.current_page + 1}"
                class="px-3 py-2 border border-gray-300 rounded hover:bg-gray-50"
              >
                Next
              </Link>
            {/if}
          </nav>
        </div>
      {/if}
    {/if}
  </main>
</div>

<!-- 9. REUSABLE COMPONENTS -->
<Footer {auth} />

<!-- 10. COMPONENT STYLES: Component-scoped CSS -->
<style>
  .prose :global(h1, h2, h3, h4, h5, h6) {
    @apply font-semibold text-gray-900 mb-4;
  }
  
  .prose :global(p) {
    @apply mb-4 text-gray-700 leading-relaxed;
  }
  
  .prose :global(a) {
    @apply text-blue-600 hover:text-blue-800 underline;
  }
  
  .content :global(img) {
    @apply max-w-full h-auto rounded-lg shadow-md my-6;
  }
</style>
```

## 🔐 Authentication Patterns

### **Login Component**
```javascript
// Complete login pattern with validation and error handling
let values = $state({ email: '', password: '', remember: false })
let processing = $state(false)
let { errors = {} } = $props()

// Client-side validation
let validation = $derived({
  email: {
    isValid: /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email),
    message: values.email.length === 0 ? '' : 
             !values.email.includes('@') ? 'Please enter a valid email address' : ''
  },
  password: {
    isValid: values.password.length >= 6,
    message: values.password.length === 0 ? '' :
             values.password.length < 6 ? 'Password must be at least 6 characters' : ''
  }
})

let canSubmit = $derived(
  validation.email.isValid && 
  validation.password.isValid && 
  !processing
)

function handleLogin(event) {
  event.preventDefault()
  if (!canSubmit) return
  
  processing = true
  
  router.post('/login', values, {
    onSuccess: () => {
      console.log('Login successful!')
    },
    onError: (errors) => {
      console.log('Login failed:', errors)
    },
    onFinish: () => {
      processing = false
    }
  })
}
```

### **Registration Component**
```javascript
let values = $state({
  name: '',
  email: '',
  password: '',
  password_confirmation: ''
})

let validation = $derived({
  name: {
    isValid: values.name.trim().length >= 2,
    message: values.name.length === 0 ? '' : 
             values.name.trim().length < 2 ? 'Name must be at least 2 characters' : ''
  },
  email: {
    isValid: /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email),
    message: values.email.length === 0 ? '' : 
             'Please enter a valid email address'
  },
  password: {
    isValid: values.password.length >= 8,
    message: values.password.length === 0 ? '' :
             values.password.length < 8 ? 'Password must be at least 8 characters' : ''
  },
  confirmation: {
    isValid: values.password_confirmation === values.password && values.password.length > 0,
    message: values.password_confirmation.length === 0 ? '' :
             values.password_confirmation !== values.password ? 'Passwords do not match' : ''
  }
})
```

### **Password Reset Flow**
```javascript
// Forgot password request
function handleForgotPassword(event) {
  event.preventDefault()
  processing = true
  
  router.post('/forgot-password', { email: values.email }, {
    onSuccess: () => {
      showSuccessMessage = true
      successMessage = 'Password reset email sent! Check your inbox.'
    },
    onError: (errors) => {
      console.log('Failed to send reset email:', errors)
    },
    onFinish: () => {
      processing = false
    }
  })
}

// Password reset with token
function handlePasswordReset(event) {
  event.preventDefault()
  processing = true
  
  router.post('/reset-password', {
    token: resetToken,
    email: values.email,
    password: values.password,
    password_confirmation: values.password_confirmation
  }, {
    onSuccess: () => {
      router.visit('/login')
    },
    onFinish: () => {
      processing = false
    }
  })
}
```

### **User Profile Management**
```javascript
let profileValues = $state({
  name: user?.name || '',
  email: user?.email || ''
})

let passwordValues = $state({
  current_password: '',
  password: '',
  password_confirmation: ''
})

function handleProfileUpdate(event) {
  event.preventDefault()
  processing = true
  
  router.put('/profile', profileValues, {
    onSuccess: () => {
      showMessage('Profile updated successfully!')
    },
    onFinish: () => {
      processing = false
    }
  })
}

function handlePasswordChange(event) {
  event.preventDefault()
  processingPassword = true
  
  router.put('/profile/password', passwordValues, {
    onSuccess: () => {
      passwordValues = {
        current_password: '',
        password: '',
        password_confirmation: ''
      }
      showMessage('Password changed successfully!')
    },
    onFinish: () => {
      processingPassword = false
    }
  })
}
```

## 📚 Blog CRUD Patterns

### **Blog Post Creation**
```javascript
let values = $state({
  title: '',
  content: '',
  meta_description: '',
  status: 'draft',
  featured: false
})

// Auto-generate slug from title
let generatedSlug = $derived(
  values.title
    .toLowerCase()
    .replace(/[^a-z0-9\s-]/g, '')
    .replace(/\s+/g, '-')
    .replace(/-+/g, '-')
    .trim('-')
)

// Auto-save as draft
let autoSaveTimer = $state(null)
$effect(() => {
  if (values.title || values.content) {
    if (autoSaveTimer) clearTimeout(autoSaveTimer)
    
    autoSaveTimer = setTimeout(() => {
      if (values.title.trim()) {
        router.post('/posts/auto-save', {
          ...values,
          slug: generatedSlug
        }, {
          preserveState: true,
          onSuccess: () => {
            console.log('Auto-saved')
          }
        })
      }
    }, 30000)
  }
  
  return () => {
    if (autoSaveTimer) clearTimeout(autoSaveTimer)
  }
})

function handleCreate(event) {
  event.preventDefault()
  processing = true
  
  router.post('/posts', {
    ...values,
    slug: generatedSlug
  }, {
    onSuccess: () => {
      values = { title: '', content: '', meta_description: '', status: 'draft', featured: false }
    },
    onFinish: () => {
      processing = false
    }
  })
}
```

### **Blog Post Editing**
```javascript
// Initialize form with existing post data
let values = $state({
  title: post?.title || '',
  content: post?.content || '',
  meta_description: post?.meta_description || '',
  status: post?.status || 'draft',
  featured: post?.featured || false
})

// Track changes
let hasChanges = $derived(
  values.title !== post?.title ||
  values.content !== post?.content ||
  values.meta_description !== post?.meta_description ||
  values.status !== post?.status ||
  values.featured !== post?.featured
)

// Warn before leaving with unsaved changes
$effect(() => {
  function handleBeforeUnload(event) {
    if (hasChanges) {
      event.preventDefault()
      event.returnValue = ''
    }
  }
  
  window.addEventListener('beforeunload', handleBeforeUnload)
  
  return () => {
    window.removeEventListener('beforeunload', handleBeforeUnload)
  }
})

function handleUpdate(event) {
  event.preventDefault()
  processing = true
  
  router.put(`/posts/${post.id}`, values, {
    onSuccess: () => {
      showMessage('Post updated successfully!')
    },
    onFinish: () => {
      processing = false
    }
  })
}
```

### **Blog Post Deletion with Confirmation**
```javascript
let showDeleteConfirm = $state(false)
let deleteConfirmText = $state('')

function handleDeleteClick(post) {
  showDeleteConfirm = true
  postToDelete = post
}

function confirmDelete() {
  if (deleteConfirmText !== 'DELETE') {
    return
  }
  
  router.delete(`/posts/${postToDelete.id}`, {
    onSuccess: () => {
      showDeleteConfirm = false
      deleteConfirmText = ''
      postToDelete = null
      showMessage('Post deleted successfully!')
    }
  })
}
```

### **Blog Search and Filtering**
```javascript
let searchQuery = $state('')
let statusFilter = $state('all')
let sortBy = $state('created_at')
let sortDirection = $state('desc')

// Debounced search
let searchTimer = $state(null)
$effect(() => {
  if (searchTimer) clearTimeout(searchTimer)
  
  searchTimer = setTimeout(() => {
    performSearch()
  }, 500)
  
  return () => {
    if (searchTimer) clearTimeout(searchTimer)
  }
})

function performSearch() {
  const params = new URLSearchParams()
  if (searchQuery) params.set('search', searchQuery)
  if (statusFilter !== 'all') params.set('status', statusFilter)
  params.set('sort', sortBy)
  params.set('direction', sortDirection)
  
  router.get(`/posts?${params.toString()}`, {}, {
    preserveState: true,
    preserveScroll: true
  })
}

function handleFilterChange(key, value) {
  if (key === 'search') searchQuery = value
  if (key === 'status') statusFilter = value
  if (key === 'sort') sortBy = value
  if (key === 'direction') sortDirection = value
  
  performSearch()
}
```

## 🌐 SEO and Metadata Patterns

### **Dynamic SEO for Blog Posts**
```svelte
<svelte:head>
  {#if post}
    <title>{post.title} - Educational Blog</title>
    <meta name="description" content={post.meta_description || post.excerpt || post.title} />
    
    <!-- Open Graph -->
    <meta property="og:title" content={post.title} />
    <meta property="og:description" content={post.meta_description || post.excerpt} />
    <meta property="og:type" content="article" />
    <meta property="og:url" content="{window.location.origin}/posts/{post.slug}" />
    {#if post.featured_image}
      <meta property="og:image" content={post.featured_image} />
    {/if}
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content={post.title} />
    <meta name="twitter:description" content={post.meta_description || post.excerpt} />
    
    <!-- Article specific -->
    <meta property="article:published_time" content={post.created_at} />
    <meta property="article:modified_time" content={post.updated_at} />
    <meta property="article:author" content={post.author?.name} />
    
    <!-- Canonical URL -->
    <link rel="canonical" href="{window.location.origin}/posts/{post.slug}" />
  {:else}
    <title>Blog Posts - Educational Blog</title>
    <meta name="description" content="Explore our collection of educational blog posts covering web development, programming, and technology." />
  {/if}
</svelte:head>
```

### **Structured Data for SEO**
```svelte
{#if post}
  <svelte:head>
    <script type="application/ld+json">
      {JSON.stringify({
        "@context": "https://schema.org",
        "@type": "BlogPosting",
        "headline": post.title,
        "description": post.meta_description || post.excerpt,
        "author": {
          "@type": "Person",
          "name": post.author?.name
        },
        "datePublished": post.created_at,
        "dateModified": post.updated_at,
        "mainEntityOfPage": {
          "@type": "WebPage",
          "@id": `${window.location.origin}/posts/${post.slug}`
        },
        "publisher": {
          "@type": "Organization",
          "name": "Educational Blog"
        }
      })}
    </script>
  </svelte:head>
{/if}
```

## 🎯 Common Patterns Summary

### **Do Use ✅**
- `router.post()`, `router.put()`, `router.delete()` for all Inertia form submissions
- `$state()` for **all reactive local state variables** (simple values, objects, arrays)
- `bind:value={yourStateVariable}` for **two-way data binding** in form inputs
- Simple, pure expressions in `$derived()` for computed values
- `onclick={handlerFunction}` or `oninput={inlineHandler}` for event handling
- Props destructuring with default values (`let { prop = defaultValue } = $props()`)
- Explicitly manage processing/loading state with `$state(false/true)`
- Use `onSuccess`, `onError`, `onFinish` callbacks in `router` calls for UX
- Client-side validation with `$derived()` for real-time feedback
- Auto-save functionality with `$effect()` and setTimeout
- Debounced search with `$effect()` cleanup
- SEO-friendly metadata with `<svelte:head>`
- Confirmation dialogs for destructive actions
- URL state management with query parameters

### **Don't Use ❌**
- `useForm()` from `@inertiajs/svelte` (causes reactivity issues with Svelte 5 runes)
- Complex arrow functions or side effects directly within `$derived()` (keep `$derived` pure)
- `new Component()` instantiation (use `mount()` for app, or import components directly)
- HTML comments `<!-- -->` inside Svelte directives or expressions
- Unmanaged side effects without proper cleanup in `$effect()`
- Direct DOM manipulation without considering Svelte's reactivity
- Form submissions without proper error handling and loading states

### **Advanced Patterns ⚡**
- **Auto-save with conflict resolution**: Save drafts automatically while detecting server-side changes
- **Optimistic updates**: Update UI immediately, then reconcile with server response
- **Real-time validation**: Combine client-side and server-side validation seamlessly
- **Pagination with state preservation**: Maintain search and filter state across page changes
- **File upload with progress**: Handle file uploads with progress bars and error recovery
- **Bulk operations**: Select multiple items and perform batch actions
- **Undo/Redo functionality**: Implement action history with state management
- **Keyboard shortcuts**: Add productivity shortcuts for power users
- **Accessibility**: ARIA labels, keyboard navigation, screen reader support
- **Performance optimization**: Lazy loading, virtual scrolling, code splitting

## 🎓 Educational Value

This comprehensive pattern collection demonstrates:

1. **Modern Full-Stack Development**: Complete application architecture from authentication to advanced CRUD operations
2. **Production-Ready Patterns**: Battle-tested solutions for real-world applications
3. **Performance Optimization**: Efficient patterns for search, pagination, and state management
4. **User Experience**: Loading states, validation feedback, confirmation dialogs, auto-save
5. **Security Best Practices**: Proper authentication, authorization, and data validation
6. **SEO and Accessibility**: Complete metadata management and inclusive design
7. **Maintainable Code**: Consistent patterns and clear documentation for team development

## 📚 References

- [Inertia.js Forms Documentation](https://inertiajs.com/forms)
- [Svelte 5 Documentation](https://svelte.dev/docs)
- [Laravel Inertia Documentation](https://inertiajs.com/)
- [Tailwind CSS v4 Documentation](https://tailwindcss.com/)

---

*This guide represents our complete journey building a production-ready full-stack application. Every pattern is tested, documented, and ready for immediate use in your projects.* 