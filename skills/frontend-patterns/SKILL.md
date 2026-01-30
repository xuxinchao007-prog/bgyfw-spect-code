---
name: frontend-patterns-vue
description: Comprehensive frontend development patterns for React/Next.js and Vue 3/Nuxt.js, including component patterns, state management, performance optimization, and UI best practices for modern frameworks.
---

# Modern Frontend Development Patterns

Comprehensive patterns for building modern, performant web applications with **React/Next.js** and **Vue 3/Nuxt.js**.

## Table of Contents

1. [Component Patterns](#component-patterns)
2. [Custom Hooks & Composables](#custom-hooks--composables)
3. [State Management](#state-management)
4. [Performance Optimization](#performance-optimization)
5. [Form Handling](#form-handling)
6. [Error Handling](#error-handling)
7. [Animation Patterns](#animation-patterns)
8. [Accessibility Patterns](#accessibility-patterns)
9. [Framework-Specific Features](#framework-specific-features)

---

## Component Patterns

### Composition Over Inheritance

#### React

```typescript
// ✅ GOOD: Component composition
interface CardProps {
  children: React.ReactNode
  variant?: 'default' | 'outlined'
}

export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}

export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}

export function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>
}

// Usage
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
</Card>
```

#### Vue 3

```vue
<!-- Card.vue -->
<script setup lang="ts">
interface CardProps {
  variant?: 'default' | 'outlined'
}

withDefaults(defineProps<CardProps>(), { variant: 'default' })
</script>

<template>
  <div :class="['card', `card-${variant}`]">
    <slot />
  </div>
</template>

<!-- CardHeader.vue -->
<script setup lang="ts">
// No props needed - just a slot container
</script>

<template>
  <div class="card-header">
    <slot />
  </div>
</template>

<!-- CardBody.vue -->
<script setup lang="ts">
// No props needed
</script>

<template>
  <div class="card-body">
    <slot />
  </div>
</template>

<!-- Usage -->
<Card>
  <CardHeader>Title</CardHeader>
  <CardBody>Content</CardBody>
</Card>
```

### Compound Components

#### React

```typescript
interface TabsContextValue {
  activeTab: string
  setActiveTab: (tab: string) => void
}

const TabsContext = createContext<TabsContextValue | undefined>(undefined)

export function Tabs({ children, defaultTab }: {
  children: React.ReactNode
  defaultTab: string
}) {
  const [activeTab, setActiveTab] = useState(defaultTab)

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      {children}
    </TabsContext.Provider>
  )
}

export function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list">{children}</div>
}

export function Tab({ id, children }: { id: string, children: React.ReactNode }) {
  const context = useContext(TabsContext)
  if (!context) throw new Error('Tab must be used within Tabs')

  return (
    <button
      className={context.activeTab === id ? 'active' : ''}
      onClick={() => context.setActiveTab(id)}
    >
      {children}
    </button>
  )
}
```

#### Vue 3 (Provide/Inject Pattern)

```vue
<!-- Tabs.vue -->
<script setup lang="ts">
import { provide, ref } from 'vue'

interface TabsContext {
  activeTab: Ref<string>
  setActiveTab: (tab: string) => void
}

const props = defineProps<{
  defaultTab: string
}>()

const activeTab = ref(props.defaultTab)

const setActiveTab = (tab: string) => {
  activeTab.value = tab
}

provide<TabsContext>('tabs', {
  activeTab,
  setActiveTab
})
</script>

<template>
  <div class="tabs">
    <slot />
  </div>
</template>

<!-- TabList.vue -->
<script setup lang="ts">
// Container for tab buttons
</script>

<template>
  <div class="tab-list">
    <slot />
  </div>
</template>

<!-- Tab.vue -->
<script setup lang="ts">
import { inject } from 'vue'

const props = defineProps<{
  id: string
}>()

const tabs = inject<TabsContext>('tabs')
if (!tabs) throw new Error('Tab must be used within Tabs')
</script>

<template>
  <button
    :class="{ active: tabs.activeTab === id }"
    @click="tabs.setActiveTab(id)"
  >
    <slot />
  </button>
</template>
```

### Render Props / Slot Pattern

#### React (Render Props)

```typescript
interface DataLoaderProps<T> {
  url: string
  children: (data: T | null, loading: boolean, error: Error | null) => React.ReactNode
}

export function DataLoader<T>({ url, children }: DataLoaderProps<T>) {
  const [data, setData] = useState<T | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [url])

  return <>{children(data, loading, error)}</>
}

// Usage
<DataLoader<Market[]> url="/api/markets">
  {(markets, loading, error) => {
    if (loading) return <Spinner />
    if (error) return <Error error={error} />
    return <MarketList markets={markets!} />
  }}
</DataLoader>
```

#### Vue 3 (Scoped Slots)

```vue
<!-- DataLoader.vue -->
<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

const props = defineProps<{
  url: string
}>()

const data = ref<T | null>(null)
const loading = ref(true)
const error = ref<Error | null>(null)

const fetchData = async () => {
  loading.value = true
  error.value = null

  try {
    const response = await fetch(props.url)
    data.value = await response.json()
  } catch (err) {
    error.value = err as Error
  } finally {
    loading.value = false
  }
}

onMounted(fetchData)
watch(() => props.url, fetchData)

defineExpose({
  data,
  loading,
  error
})
</script>

<template>
  <slot :data="data" :loading="loading" :error="error" />
</template>

<!-- Usage -->
<DataLoader url="/api/markets">
  <template #default="{ data: markets, loading, error }">
    <Spinner v-if="loading" />
    <Error v-else-if="error" :error="error" />
    <MarketList v-else :markets="markets" />
  </template>
</DataLoader>
```

---

## Custom Hooks & Composables

### State Management Hook / Composable

#### React (useToggle)

```typescript
export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)

  const toggle = useCallback(() => {
    setValue(v => !v)
  }, [])

  return [value, toggle]
}

// Usage
const [isOpen, toggleOpen] = useToggle()
```

#### Vue 3 (useToggle)

```typescript
// composables/useToggle.ts
import { ref } from 'vue'

export function useToggle(initialValue = false) {
  const value = ref(initialValue)

  const toggle = () => {
    value.value = !value.value
  }

  return {
    value,
    toggle
  }
}

// Usage in component
<script setup lang="ts">
const { value: isOpen, toggle } = useToggle()
</script>
```

### Async Data Fetching Hook / Composable

#### React (useQuery)

```typescript
interface UseQueryOptions<T> {
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
  enabled?: boolean
}

export function useQuery<T>(
  key: string,
  fetcher: () => Promise<T>,
  options?: UseQueryOptions<T>
) {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [loading, setLoading] = useState(false)

  const refetch = useCallback(async () => {
    setLoading(true)
    setError(null)

    try {
      const result = await fetcher()
      setData(result)
      options?.onSuccess?.(result)
    } catch (err) {
      const error = err as Error
      setError(error)
      options?.onError?.(error)
    } finally {
      setLoading(false)
    }
  }, [fetcher, options])

  useEffect(() => {
    if (options?.enabled !== false) {
      refetch()
    }
  }, [key, refetch, options?.enabled])

  return { data, error, loading, refetch }
}

// Usage
const { data: markets, loading, error, refetch } = useQuery(
  'markets',
  () => fetch('/api/markets').then(r => r.json()),
  {
    onSuccess: data => console.log('Fetched', data.length, 'markets'),
    onError: err => console.error('Failed:', err)
  }
)
```

#### Vue 3 (useFetch)

```typescript
// composables/useFetch.ts
import { ref, onMounted } from 'vue'

interface UseFetchOptions<T> {
  onSuccess?: (data: T) => void
  onError?: (error: Error) => void
  immediate?: boolean
}

export function useFetch<T>(
  fetcher: () => Promise<T>,
  options?: UseFetchOptions<T>
) {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(false)

  const execute = async () => {
    loading.value = true
    error.value = null

    try {
      const result = await fetcher()
      data.value = result
      options?.onSuccess?.(result)
    } catch (err) {
      const errorValue = err as Error
      error.value = errorValue
      options?.onError?.(errorValue)
    } finally {
      loading.value = false
    }
  }

  if (options?.immediate !== false) {
    onMounted(execute)
  }

  return {
    data,
    error,
    loading,
    execute
  }
}

// Usage
<script setup lang="ts">
const { data: markets, loading, error, execute } = useFetch(
  () => fetch('/api/markets').then(r => r.json()),
  {
    onSuccess: data => console.log('Fetched', data.length, 'markets'),
    onError: err => console.error('Failed:', err)
  }
)
</script>
```

### Debounce Hook / Composable

#### React (useDebounce)

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// Usage
const [searchQuery, setSearchQuery] = useState('')
const debouncedQuery = useDebounce(searchQuery, 500)

useEffect(() => {
  if (debouncedQuery) {
    performSearch(debouncedQuery)
  }
}, [debouncedQuery])
```

#### Vue 3 (useDebounce)

```typescript
// composables/useDebounce.ts
import { ref, watch } from 'vue'

export function useDebounce<T>(value: Ref<T>, delay: number) {
  const debouncedValue = ref(value.value) as Ref<T>

  let timeout: ReturnType<typeof setTimeout> | null = null

  watch(value, (newValue) => {
    if (timeout) clearTimeout(timeout)

    timeout = setTimeout(() => {
      debouncedValue.value = newValue
    }, delay)
  })

  return debouncedValue
}

// Usage
<script setup lang="ts">
const searchQuery = ref('')
const debouncedQuery = useDebounce(searchQuery, 500)

watch(debouncedQuery, (query) => {
  if (query) {
    performSearch(query)
  }
})
</script>
```

---

## State Management

### Context + Reducer Pattern (React)

```typescript
interface State {
  markets: Market[]
  selectedMarket: Market | null
  loading: boolean
}

type Action =
  | { type: 'SET_MARKETS'; payload: Market[] }
  | { type: 'SELECT_MARKET'; payload: Market }
  | { type: 'SET_LOADING'; payload: boolean }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'SET_MARKETS':
      return { ...state, markets: action.payload }
    case 'SELECT_MARKET':
      return { ...state, selectedMarket: action.payload }
    case 'SET_LOADING':
      return { ...state, loading: action.payload }
    default:
      return state
  }
}

const MarketContext = createContext<{
  state: State
  dispatch: Dispatch<Action>
} | undefined>(undefined)

export function MarketProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(reducer, {
    markets: [],
    selectedMarket: null,
    loading: false
  })

  return (
    <MarketContext.Provider value={{ state, dispatch }}>
      {children}
    </MarketContext.Provider>
  )
}

export function useMarkets() {
  const context = useContext(MarketContext)
  if (!context) throw new Error('useMarkets must be used within MarketProvider')
  return context
}
```

### Provide/Inject + Reactive Store (Vue 3)

```typescript
// composables/useMarketStore.ts
import { provide, inject, reactive, computed } from 'vue'

const MARKET_STORE_KEY = Symbol('marketStore')

interface MarketState {
  markets: Market[]
  selectedMarket: Market | null
  loading: boolean
}

interface MarketActions {
  setMarkets: (markets: Market[]) => void
  selectMarket: (market: Market) => void
  setLoading: (loading: boolean) => void
}

type MarketStore = MarketState & MarketActions

export function provideMarketStore() {
  const state = reactive<MarketState>({
    markets: [],
    selectedMarket: null,
    loading: false
  })

  const actions: MarketActions = {
    setMarkets: (markets) => {
      state.markets = markets
    },
    selectMarket: (market) => {
      state.selectedMarket = market
    },
    setLoading: (loading) => {
      state.loading = loading
    }
  }

  const store = computed(() => ({
    ...state,
    ...actions
  }))

  provide(MARKET_STORE_KEY, store)

  return store
}

export function useMarketStore() {
  const store = inject<ComputedRef<MarketStore>>(MARKET_STORE_KEY)

  if (!store) {
    throw new Error('useMarketStore must be used within provideMarketStore')
  }

  return store.value
}

// Usage in app
<script setup lang="ts">
provideMarketStore()
</script>

<!-- Usage in component -->
<script setup lang="ts">
const { markets, selectedMarket, setMarkets, selectMarket } = useMarketStore()
</script>
```

### Pinia Store (Vue 3 - Recommended)

```typescript
// stores/marketStore.ts
import { defineStore } from 'pinia'

export const useMarketStore = defineStore('market', {
  state: () => ({
    markets: [] as Market[],
    selectedMarket: null as Market | null,
    loading: false
  }),

  getters: {
    activeMarkets: (state) => state.markets.filter(m => m.active),
    marketCount: (state) => state.markets.length
  },

  actions: {
    async fetchMarkets() {
      this.loading = true
      try {
        const response = await fetch('/api/markets')
        this.markets = await response.json()
      } finally {
        this.loading = false
      }
    },

    selectMarket(market: Market) {
      this.selectedMarket = market
    }
  }
})

// Usage in component
<script setup lang="ts">
const marketStore = useMarketStore()

const { markets, selectedMarket, loading } = storeToRefs(marketStore)
const { fetchMarkets, selectMarket } = marketStore

onMounted(() => {
  fetchMarkets()
})
</script>
```

### Zustand Store (React - Lightweight Alternative)

```typescript
// stores/marketStore.ts
import create from 'zustand'

interface MarketStore {
  markets: Market[]
  selectedMarket: Market | null
  loading: boolean
  setMarkets: (markets: Market[]) => void
  selectMarket: (market: Market) => void
  setLoading: (loading: boolean) => void
  fetchMarkets: () => Promise<void>
}

export const useMarketStore = create<MarketStore>((set, get) => ({
  markets: [],
  selectedMarket: null,
  loading: false,

  setMarkets: (markets) => set({ markets }),
  selectMarket: (market) => set({ selectedMarket: market }),
  setLoading: (loading) => set({ loading }),

  fetchMarkets: async () => {
    set({ loading: true })
    try {
      const response = await fetch('/api/markets')
      set({ markets: await response.json() })
    } finally {
      set({ loading: false })
    }
  }
}))

// Usage
function MarketList() {
  const { markets, loading, fetchMarkets } = useMarketStore()

  useEffect(() => {
    fetchMarkets()
  }, [])

  if (loading) return <Spinner />
  return <div>{markets.map(m => <MarketCard key={m.id} market={m} />)}</div>
}
```

---

## Performance Optimization

### Memoization

#### React

```typescript
// ✅ useMemo for expensive computations
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ useCallback for functions passed to children
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])

// ✅ React.memo for pure components
export const MarketCard = React.memo<MarketCardProps>(({ market }) => {
  return (
    <div className="market-card">
      <h3>{market.name}</h3>
      <p>{market.description}</p>
    </div>
  )
}, (prevProps, nextProps) => {
  // Custom comparison
  return prevProps.market.id === nextProps.market.id
})
```

#### Vue 3 (Computed + Watch)

```vue
<script setup lang="ts">
import { computed, watch } from 'vue'

const props = defineProps<{
  markets: Market[]
}>()

// ✅ Computed for derived state (memoized by default)
const sortedMarkets = computed(() => {
  return [...props.markets].sort((a, b) => b.volume - a.volume)
})

// ✅ Watch for side effects
watch(
  () => props.markets,
  (newMarkets) => {
    console.log('Markets updated:', newMarkets.length)
  },
  { deep: true }
)
</script>
```

### Code Splitting & Lazy Loading

#### React

```typescript
import { lazy, Suspense } from 'react'

// ✅ Lazy load heavy components
const HeavyChart = lazy(() => import('./HeavyChart'))
const ThreeJsBackground = lazy(() => import('./ThreeJsBackground'))

export function Dashboard() {
  return (
    <div>
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart data={data} />
      </Suspense>

      <Suspense fallback={null}>
        <ThreeJsBackground />
      </Suspense>
    </div>
  )
}
```

#### Vue 3

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

// ✅ Async component with loading and error states
const HeavyChart = defineAsyncComponent({
  loader: () => import('./HeavyChart.vue'),
  loadingComponent: ChartSkeleton,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})

const ThreeJsBackground = defineAsyncComponent(() =>
  import('./ThreeJsBackground.vue')
)
</script>

<template>
  <div>
    <Suspense>
      <template #default>
        <HeavyChart :data="data" />
      </template>
      <template #fallback>
        <ChartSkeleton />
      </template>
    </Suspense>

    <Suspense>
      <ThreeJsBackground />
      <template #fallback>
        <div>Loading background...</div>
      </template>
    </Suspense>
  </div>
</template>
```

### Virtualization for Long Lists

#### React

```typescript
import { useVirtualizer } from '@tanstack/react-virtual'

export function VirtualMarketList({ markets }: { markets: Market[] }) {
  const parentRef = useRef<HTMLDivElement>(null)

  const virtualizer = useVirtualizer({
    count: markets.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 100,
    overscan: 5
  })

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map(virtualRow => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`
            }}
          >
            <MarketCard market={markets[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  )
}
```

#### Vue 3

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useVirtualList } from '@vueuse/core'

const props = defineProps<{
  markets: Market[]
}>()

const containerRef = ref<HTMLElement | null>(null)

const { list, containerProps, wrapperProps } = useVirtualList(
  props.markets,
  {
    itemHeight: 100,
    overscan: 5
  }
)
</script>

<template>
  <div
    ref="containerRef"
    v-bind="containerProps"
    style="{ height: '600px', overflow: 'auto' }"
  >
    <div v-bind="wrapperProps">
      <div
        v-for="{ data: market, index } in list"
        :key="market.id"
        style="{ height: '100px' }"
      >
        <MarketCard :market="market" />
      </div>
    </div>
  </div>
</template>
```

### Next.js App Router Optimization

```typescript
// ✅ Dynamic imports for client-side only components
'use client'

import dynamic from 'next/dynamic'

const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false // Disable SSR for this component
})

// ✅ Streaming with Suspense
export default async function DashboardPage() {
  const data = await fetchMarkets() // Server component

  return (
    <div>
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart data={data} />
      </Suspense>
    </div>
  )
}
```

### Nuxt 3 Optimization

```vue
<!-- Lazy loading with Nuxt -->
<script setup lang="ts">
// ✅ Nuxt auto-imports and lazy loads components from ~/components
const HeavyChart = await resolveComponent('HeavyChart')
</script>

<template>
  <div>
    <!-- ✅ Lazy hydration with Lazy wrapper -->
    <LazyHeavyChart v-if="showChart" :data="data" />
  </div>
</template>
```

---

## Form Handling

### Controlled Form with Validation

#### React

```typescript
interface FormData {
  name: string
  description: string
  endDate: string
}

interface FormErrors {
  name?: string
  description?: string
  endDate?: string
}

export function CreateMarketForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    description: '',
    endDate: ''
  })

  const [errors, setErrors] = useState<FormErrors>({})

  const validate = (): boolean => {
    const newErrors: FormErrors = {}

    if (!formData.name.trim()) {
      newErrors.name = 'Name is required'
    } else if (formData.name.length > 200) {
      newErrors.name = 'Name must be under 200 characters'
    }

    if (!formData.description.trim()) {
      newErrors.description = 'Description is required'
    }

    if (!formData.endDate) {
      newErrors.endDate = 'End date is required'
    }

    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()

    if (!validate()) return

    try {
      await createMarket(formData)
    } catch (error) {
      // Handle error
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
        placeholder="Market name"
      />
      {errors.name && <span className="error">{errors.name}</span>}
      <button type="submit">Create Market</button>
    </form>
  )
}
```

#### Vue 3 (with VeeValidate)

```vue
<script setup lang="ts">
import { useForm } from 'vee-validate'
import * as yup from 'yup'

const schema = yup.object({
  name: yup.string()
    .required('Name is required')
    .max(200, 'Name must be under 200 characters'),
  description: yup.string()
    .required('Description is required'),
  endDate: yup.string()
    .required('End date is required')
})

const { defineField, handleSubmit, errors } = useForm({
  validationSchema: schema
})

const [name] = defineField('name')
const [description] = defineField('description')
const [endDate] = defineField('endDate')

const onSubmit = handleSubmit(async (values) => {
  try {
    await createMarket(values)
  } catch (error) {
    // Handle error
  }
})
</script>

<template>
  <form @submit="onSubmit">
    <input
      v-model="name"
      placeholder="Market name"
    />
    <span v-if="errors.name" class="error">{{ errors.name }}</span>

    <button type="submit">Create Market</button>
  </form>
</template>
```

---

## Error Handling

### Error Boundary (React)

```typescript
interface ErrorBoundaryState {
  hasError: boolean
  error: Error | null
}

export class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ReactNode },
  ErrorBoundaryState
> {
  state: ErrorBoundaryState = {
    hasError: false,
    error: null
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error boundary caught:', error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      )
    }

    return this.props.children
  }
}
```

### Error Handler (Vue 3)

```typescript
// plugins/errorHandler.ts
import type { App } from 'vue'

export function setupErrorHandler(app: App) {
  app.config.errorHandler = (err, instance, info) => {
    console.error('Vue error:', err)
    console.error('Component:', instance)
    console.error('Info:', info)

    // Send to error tracking service
    // trackError(err, { instance, info })
  }
}

// main.ts
const app = createApp(App)
setupErrorHandler(app)
```

### Error Boundary Component (Vue 3)

```vue
<!-- ErrorBoundary.vue -->
<script setup lang="ts">
import { ref, onErrorCaptured } from 'vue'

const props = defineProps<{
  fallback?: Component
}>()

const hasError = ref(false)
const error = ref<Error | null>(null)

onErrorCaptured((err) => {
  hasError.value = true
  error.value = err
  console.error('Error boundary caught:', err)

  // Prevent error from propagating
  return false
})

const reset = () => {
  hasError.value = false
  error.value = null
}
</script>

<template>
  <slot v-if="!hasError" />
  <component
    v-else-if="fallback"
    :is="fallback"
    :error="error"
    @reset="reset"
  />
  <div v-else class="error-fallback">
    <h2>Something went wrong</h2>
    <p>{{ error?.message }}</p>
    <button @click="reset">Try again</button>
  </div>
</template>
```

---

## Animation Patterns

### Framer Motion (React)

```typescript
import { motion, AnimatePresence } from 'framer-motion'

// ✅ List animations
export function AnimatedMarketList({ markets }: { markets: Market[] }) {
  return (
    <AnimatePresence>
      {markets.map(market => (
        <motion.div
          key={market.id}
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -20 }}
          transition={{ duration: 0.3 }}
        >
          <MarketCard market={market} />
        </motion.div>
      ))}
    </AnimatePresence>
  )
}

// ✅ Modal animations
export function Modal({ isOpen, onClose, children }: ModalProps) {
  return (
    <AnimatePresence>
      {isOpen && (
        <>
          <motion.div
            className="modal-overlay"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            onClick={onClose}
          />
          <motion.div
            className="modal-content"
            initial={{ opacity: 0, scale: 0.9, y: 20 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.9, y: 20 }}
          >
            {children}
          </motion.div>
        </>
      )}
    </AnimatePresence>
  )
}
```

### Vue 3 Transitions

```vue
<!-- Animated List -->
<script setup lang="ts">
const props = defineProps<{
  markets: Market[]
}>()
</script>

<template>
  <TransitionGroup
    name="market"
    tag="div"
  >
    <div
      v-for="market in markets"
      :key="market.id"
      class="market-item"
    >
      <MarketCard :market="market" />
    </div>
  </TransitionGroup>
</template>

<style scoped>
.market-enter-active,
.market-leave-active {
  transition: all 0.3s ease;
}

.market-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.market-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

.market-move {
  transition: transform 0.3s ease;
}
</style>
```

### Vue 3 Transition Component

```vue
<!-- Modal.vue -->
<script setup lang="ts">
const props = defineProps<{
  isOpen: boolean
}>()

const emit = defineEmits<{
  close: []
}>()
</script>

<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen" class="modal-wrapper">
        <div class="modal-overlay" @click="emit('close')" />
        <div class="modal-content">
          <slot />
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<style scoped>
.modal-enter-active,
.modal-leave-active {
  transition: all 0.3s ease;
}

.modal-enter-from .modal-content,
.modal-leave-to .modal-content {
  opacity: 0;
  transform: scale(0.9) translateY(20px);
}

.modal-enter-from .modal-overlay,
.modal-leave-to .modal-overlay {
  opacity: 0;
}
</style>
```

---

## Accessibility Patterns

### Keyboard Navigation

#### React

```typescript
export function Dropdown({ options, onSelect }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false)
  const [activeIndex, setActiveIndex] = useState(0)

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault()
        setActiveIndex(i => Math.min(i + 1, options.length - 1))
        break
      case 'ArrowUp':
        e.preventDefault()
        setActiveIndex(i => Math.max(i - 1, 0))
        break
      case 'Enter':
        e.preventDefault()
        onSelect(options[activeIndex])
        setIsOpen(false)
        break
      case 'Escape':
        setIsOpen(false)
        break
    }
  }

  return (
    <div
      role="combobox"
      aria-expanded={isOpen}
      aria-haspopup="listbox"
      onKeyDown={handleKeyDown}
    >
      {/* Dropdown implementation */}
    </div>
  )
}
```

#### Vue 3

```vue
<script setup lang="ts">
const props = defineProps<{
  options: Option[]
}>()

const emit = defineEmits<{
  select: [option: Option]
}>()

const isOpen = ref(false)
const activeIndex = ref(0)

const handleKeyDown = (e: KeyboardEvent) => {
  switch (e.key) {
    case 'ArrowDown':
      e.preventDefault()
      activeIndex.value = Math.min(activeIndex.value + 1, props.options.length - 1)
      break
    case 'ArrowUp':
      e.preventDefault()
      activeIndex.value = Math.max(activeIndex.value - 1, 0)
      break
    case 'Enter':
      e.preventDefault()
      emit('select', props.options[activeIndex.value])
      isOpen.value = false
      break
    case 'Escape':
      isOpen.value = false
      break
  }
}
</script>

<template>
  <div
    role="combobox"
    :aria-expanded="isOpen"
    aria-haspopup="listbox"
    @keydown="handleKeyDown"
  >
    <!-- Dropdown implementation -->
  </div>
</template>
```

### Focus Management

#### React

```typescript
export function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null)
  const previousFocusRef = useRef<HTMLElement | null>(null)

  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement as HTMLElement
      modalRef.current?.focus()
    } else {
      previousFocusRef.current?.focus()
    }
  }, [isOpen])

  return isOpen ? (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
      onKeyDown={e => e.key === 'Escape' && onClose()}
    >
      {children}
    </div>
  ) : null
}
```

#### Vue 3

```vue
<script setup lang="ts">
const props = defineProps<{
  isOpen: boolean
}>()

const emit = defineEmits<{
  close: []
}>()

const modalRef = ref<HTMLElement | null>(null)
const previousFocusRef = ref<HTMLElement | null>(null)

watch(() => props.isOpen, (isOpen) => {
  if (isOpen) {
    previousFocusRef.value = document.activeElement as HTMLElement
    nextTick(() => {
      modalRef.value?.focus()
    })
  } else {
    previousFocusRef.value?.focus()
  }
})
</script>

<template>
  <div
    v-if="isOpen"
    ref="modalRef"
    role="dialog"
    aria-modal="true"
    tabindex="-1"
    @keydown.escape="emit('close')"
  >
    <slot />
  </div>
</template>
```

---

## Framework-Specific Features

### Next.js App Router

#### Server Components

```typescript
// Server Component (runs on server by default)
async function MarketList() {
  const markets = await fetchMarkets()

  return (
    <div>
      {markets.map(market => (
        <MarketCard key={market.id} market={market} />
      ))}
    </div>
  )
}

// Client Component (for interactivity)
'use client'

export function MarketFilter() {
  const [filter, setFilter] = useState('')

  return (
    <input
      value={filter}
      onChange={e => setFilter(e.target.value)}
      placeholder="Filter markets..."
    />
  )
}
```

#### Route Handlers

```typescript
// app/api/markets/route.ts
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const markets = await getMarkets()
  return NextResponse.json(markets)
}

export async function POST(request: Request) {
  const data = await request.json()
  const market = await createMarket(data)
  return NextResponse.json(market, { status: 201 })
}
```

### Nuxt 3

#### Server-Side Rendering & Async Data

```vue
<script setup lang="ts">
// ✅ useFetch for component-level data fetching
const { data: markets, pending, error } = await useFetch<Market[]>('/api/markets')

// ✅ useAsyncData for more control
const { data: markets, refresh } = await useAsyncData(
  'markets',
  () => $fetch<Market[]>('/api/markets')
)
</script>

<template>
  <div>
    <div v-if="pending">Loading...</div>
    <div v-else-if="error">Error: {{ error.message }}</div>
    <div v-else>
      <MarketCard
        v-for="market in markets"
        :key="market.id"
        :market="market"
      />
    </div>
  </div>
</template>
```

#### Server API Routes

```typescript
// server/api/markets.get.ts
export default defineEventHandler(async (event) => {
  const markets = await getMarkets()
  return markets
})

// server/api/markets.post.ts
export default defineEventHandler(async (event) => {
  const body = await readBody(event)
  const market = await createMarket(body)
  return market
})
```

### React Server Components vs Nuxt Server Components

| Feature | Next.js RSC | Nuxt 3 |
|---------|-------------|--------|
| Default Mode | Server Components | Server-Side Rendered |
| Client Marker | `'use client'` | `<ClientOnly>` wrapper |
| Data Fetching | Direct `await` | `useFetch`, `useAsyncData` |
| Streaming | Built-in | Built-in |
| Islands Architecture | Supported | Supported (via ClientOnly) |

---

## Best Practices Summary

### React/Next.js

- Use **Server Components** by default in Next.js App Router
- Implement **Error Boundaries** for graceful error handling
- Leverage **React.memo**, **useMemo**, **useCallback** for performance
- Use **Suspense** for loading states and code splitting
- Implement **compound components** for complex UI patterns

### Vue 3/Nuxt

- Use **Composition API** with `<script setup>` for better TypeScript support
- Leverage **computed** properties (auto-memoized)
- Use **Pinia** for global state management
- Implement **Teleport** for modals and tooltips
- Use **Transition** and **TransitionGroup** for animations
- Take advantage of **Nuxt's auto-imports** for components and composables

### Cross-Framework Principles

- **Composition over inheritance** - Build flexible, reusable components
- **Separation of concerns** - Keep business logic separate from UI
- **TypeScript** - Use strict typing for better developer experience
- **Accessibility** - Ensure keyboard navigation and screen reader support
- **Performance** - Implement lazy loading, code splitting, and virtualization
- **Error boundaries** - Gracefully handle errors at component boundaries

---

**Remember**: Choose the framework and patterns that best fit your project requirements. Both React/Next.js and Vue 3/Nuxt provide excellent developer experiences and production-ready solutions for modern web applications.
