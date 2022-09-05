# its-fine

[![Size](https://img.shields.io/bundlephobia/minzip/its-fine?label=gzip&style=flat&colorA=000000&colorB=000000)](https://bundlephobia.com/package/its-fine)
[![Version](https://img.shields.io/npm/v/its-fine?style=flat&colorA=000000&colorB=000000)](https://npmjs.com/package/its-fine)
[![Twitter](https://img.shields.io/twitter/follow/pmndrs?label=%40pmndrs&style=flat&colorA=000000&colorB=000000&logo=twitter&logoColor=000000)](https://twitter.com/pmndrs)
[![Discord](https://img.shields.io/discord/740090768164651008?style=flat&colorA=000000&colorB=000000&label=discord&logo=discord&logoColor=000000)](https://discord.gg/poimandres)

<p align="left">
  <a id="cover" href="#cover">
    <img src=".github/itsfine.jpg" alt="It's gonna be alright" />
  </a>
</p>

A collection of escape hatches exploring `React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` and React Fiber. As such, you can go beyond React's component abstraction; components are self-aware and can tap into the Fiber tree. This enables powerful abstractions like stateless queries and sharing React Context across concurrent renderers.

## Table of Contents

- [Installation](#installation)
- [Hooks](#hooks)
  - [useFiber](#useFiber)
  - [useContainer](#useContainer)
  - [useNearestChild](#useNearestChild)
  - [useNearestParent](#useNearestParent)
  - [useContextBridge](#useContextBridge)
- [Utils](#utils)
  - [traverseFiber](#traverseFiber)

## Installation

```bash
# NPM
npm install its-fine
# Yarn
yarn add its-fine
# PNPM
pnpm add its-fine
```

## Hooks

Useful React hook abstractions for manipulating and querying from a component.

### useFiber

Returns the current react-internal `Fiber`. This is an implementation detail of [react-reconciler](https://github.com/facebook/react/tree/main/packages/react-reconciler).

```tsx
import * as React from 'react'
import { type Fiber, useFiber } from 'its-fine'

function Component() {
  // Returns the current component's react-internal Fiber
  const fiber: Fiber<null> = useFiber()

  // function Component() {}
  console.log(fiber.type)
}
```

### useContainer

Returns the current react-reconciler container info passed to `Reconciler.createContainer`.

In react-dom, a container will point to the root DOM element; in react-three-fiber, it will point to the root Zustand store.

```tsx
import * as React from 'react'
import { useContainer } from 'its-fine'

function Component() {
  // Returns the current renderer's root container
  const container: HTMLDivElement = useContainer<HTMLDivElement>()

  // <div> (e.g. react-dom)
  console.log(container)
}
```

### useNearestChild

Returns the nearest react-reconciler child instance or the node created from `Reconciler.createInstance`.

In react-dom, this would be a DOM element; in react-three-fiber this would be an `Instance` descriptor.

```tsx
import * as React from 'react'
import { useNearestChild } from 'its-fine'

function Component() {
  // Returns a React Ref which points to the nearest child element
  const childRef: React.MutableRefObject<HTMLDivElement | undefined> = useNearestChild<HTMLDivElement>()

  // Access child Ref on mount
  React.useEffect(() => {
    // <div> (e.g. react-dom)
    const child = childRef.current
    if (child) console.log(child)
  }, [])

  // A child element, can live deep down another component
  return <div />
}
```

### useNearestParent

Returns the nearest react-reconciler parent instance or the node created from `Reconciler.createInstance`.

In react-dom, this would be a DOM element; in react-three-fiber this would be an instance descriptor.

```tsx
import * as React from 'react'
import { useNearestParent } from 'its-fine'

function Component() {
  // Returns a React Ref which points to the nearest parent element
  const parentRef: React.MutableRefObject<HTMLDivElement | undefined> = useNearestParent<HTMLDivElement>()

  // Access parent Ref on mount
  React.useEffect(() => {
    // <div> (e.g. react-dom)
    const parent = parentRef.current
    if (parent) console.log(parent)
  }, [])
}

// A parent element wrapping Component, can live deep up another component
export default () => (
  <div>
    <Component />
  </div>
)
```

### useContextBridge

React Context currently cannot be shared across [React renderers](https://reactjs.org/docs/codebase-overview.html#renderers) but explicitly forwarded between providers (see [react#17275](https://github.com/facebook/react/issues/17275)). This hook returns a `ContextBridge` of live context providers to pierce Context across renderers.

Pass `ContextBridge` as a component to a secondary renderer to enable context-sharing within its children.

```tsx
import * as React from 'react'
// react-nil is a secondary renderer that is usually used for testing.
// This also includes Fabric, react-three-fiber, etc
import * as ReactNil from 'react-nil'
// react-dom is a primary renderer that works on top of a secondary renderer.
// This also includes react-native, react-pixi, etc.
import * as ReactDOM from 'react-dom/client'
import { type ContextBridge, useContextBridge } from 'its-fine'

function Canvas(props: { children: React.ReactNode }) {
  // Returns a bridged context provider that forwards context
  const Bridge: ContextBridge = useContextBridge()
  // Renders children with bridged context into a secondary renderer
  ReactNil.render(<Bridge>{props.children}</Bridge>)
}

// A React Context whose provider lives in react-dom
const DOMContext = React.createContext<string>(null!)

// A component that reads from DOMContext
function Component() {
  // "Hello from react-dom"
  console.log(React.useContext(DOMContext))
}

// Renders into a primary renderer like react-dom or react-native,
// DOMContext wraps Canvas and is bridged into Component
ReactDOM.createRoot(document.getElementById('root')!).render(
  <DOMContext.Provider value="Hello from react-dom">
    <Canvas>
      <Component />
    </Canvas>
  </DOMContext.Provider>,
)
```

## Utils

Additional exported utility functions for raw handling of Fibers.

### traverseFiber

Traverses up or down through a `Fiber`, return `true` to stop and select a node.

```ts
import { type Fiber, traverseFiber } from 'its-fine'

// Whether to ascend and walk up the tree. Will walk down if `false`
const ascending: boolean = true

// Traverses through the Fiber tree, returns the current node when `true` is passed via selector
const parentDiv: Fiber<HTMLDivElement> | undefined = traverseFiber<HTMLDivElement>(
  // A fiber from a composite component from `useFiber` or internally in react-reconciler
  fiber as Fiber<null>,
  ascending,
  // A Fiber node selector, returns the first <div /> element in JSX
  (node: Fiber<HTMLDivElement | null>) => node.type === 'div',
)
```
