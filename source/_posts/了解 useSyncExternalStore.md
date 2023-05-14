---
title: 了解 useSyncExternalStore
date: 2023-05-14
---

`useSyncExternalStore` 是一个 React hook，可以让你订阅一个外部存储。

## 引用

`useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

调用组件顶层的 `useSyncExternalStore` 从外部数据存储读取值。

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回存储中数据的快照。您需要传递两个函数作为参数：

1. `subscribe` 函数订阅存储并返回一个取消订阅的函数。
2. `getSnapshot` 函数从存储区读取数据的快照。

### 参数

- `subscribe`: 一个接受单个 `callback` 参数并将其订阅到存储的函数。当存储更改时，它应该调用提供的 `callback`。这将导致组件重新渲染。`subscribe` 函数应返回一个清除订阅的函数。
- `getSnapshot`: 一个函数，返回组件所需的存储数据的快照。只要存储没有改变，重复调用 `getSnapshot` 必须返回相同的值。如果存储更改并且返回值不同（通过 `Object.is` 比较），React 将重新渲染组件。
- optional `getServerSnapshot`: 一个返回存储中数据初始快照的函数。它仅在服务器渲染期间和客户端上对服务器呈现内容进行水合作用时使用。服务器快照必须在客户端和服务器之间相同，并且通常是序列化并从服务器传递到客户端。如果省略此参数，则在服务器上呈现组件将引发错误。

### 返回值

您可以在渲染逻辑中使用当前快照的 `store`。

### 说明

- 由 `getSnapshot` 返回的存储快照必须是不可变的。如果底层存储具有可变数据，则在数据更改时返回新的不可变快照。否则，返回缓存的最后一个快照。
- 如果在重新渲染期间传递了不同的 `subscribe` 函数，React将使用新传递的 `subscribe` 函数重新订阅 `store`。您可以通过在组件外部声明 `subscribe` 来防止这种情况发生。

## 使用

### 订阅外部存储

大多数 React 组件只会从它们的 `props`、`state` 和 `context` 读取数据。然而，有时一个组件需要从 React 之外的一些存储中读取一些随时间变化的数据。这包括：

- 在 React 之外保存状态的第三方状态管理库。
- 公开可变值和事件以订阅其更改的浏览器 API。

在组件的顶层调用 `useSyncExternalStore` 以从外部数据存储中读取值。

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回存储中数据的 `snapshot`。您需要传递两个函数作为参数：

1. `subscribe` 函数应该订阅商店并返回一个取消订阅的函数。
2. `getSnapshot` 函数应该从存储中读取数据的快照。

例如，在下面的沙箱中， `todosStore` 被实现为在 React 之外存储数据的外部存储。 `TodosApp` 组件使用 `useSyncExternalStore` Hook 连接到该外部存储。

[codesandbox](https://codesandbox.io/s/e3r3d3?file=/todoStore.js&utm_medium=sandpack)

### 订阅浏览器 API 

添加 `useSyncExternalStore` 的另一个原因是当您想要订阅浏览器公开的随时间变化的某些值时。例如，假设您希望您的组件显示网络连接是否处于活动状态。浏览器通过名为 `navigator.onLine` 的属性公开此信息。

这个值可以在 React 不知情的情况下改变，所以你应该用 `useSyncExternalStore` 来阅读它。

```jsx
import { useSyncExternalStore } from 'react';

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}
```

要实现 `getSnapshot` 函数，请从浏览器 API 读取当前值：

```jsx
function getSnapshot() {
  return navigator.onLine;
}
```

接下来，您需要实现 `subscribe` 函数。例如，当 `navigator.onLine` 发生变化时，浏览器会在 `window` 对象上触发 `online` 和 `offline` 事件。你需要订阅 `callback` 参数对应的事件，然后返回一个清理订阅的函数：

```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

现在 React 知道如何从外部 `navigator.onLine` API 读取值以及如何订阅它的更改。断开您的设备与网络的连接并注意组件重新呈现以响应：

```jsx
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

### 添加对服务器渲染的支持

如果您的 React 应用程序使用服务器渲染，您的 React 组件也会在浏览器环境之外运行以生成初始 HTML。这在连接到外部商店时带来了一些挑战：

- 如果您连接到仅限浏览器的 API，它将无法工作，因为它不存在于服务器上。
- 如果您连接到第三方数据存储，则需要其数据在服务器和客户端之间匹配。


要解决这些问题，请将 `getServerSnapshot` 函数作为第三个参数传递给 `useSyncExternalStore` ：

```jsx
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // Always show "Online" for server-generated HTML
}

function subscribe(callback) {
  // ...
}
```

`getServerSnapshot` 功能类似于 `getSnapshot` ，但它只在两种情况下运行：

- 它在生成 HTML 时在服务器上运行。
- 它在 hydration 期间在客户端上运行，即当 React 获取服务器 HTML 并使其交互时。

这使您可以提供初始快照值，该值将在应用程序变为交互式之前使用。如果服务器渲染没有有意义的初始值，则省略此参数以强制在客户端进行渲染。

## 故障排查

### `I’m getting an error: “The result of getSnapshot should be cached”`

这个错误意味着你的 `getSnapshot` 函数每次被调用时都会返回一个新对象，例如：

```jsx
function getSnapshot() {
  // 🔴 Do not return always different objects from getSnapshot
  return {
    todos: myStore.todos
  };
}
```

您的 `getSnapshot` 对象应该只在实际发生更改时返回一个不同的对象。如果您的商店包含不可变数据，您可以直接返回该数据：

```jsx
function getSnapshot() {
  // ✅ You can return immutable data
  return myStore.todos;
}
```

如果您的 `store` 数据是可变的，您的 `getSnapshot` 函数应该返回它的不可变快照。这意味着它确实需要创建新对象，但它不应该为每次调用都这样做。相反，它应该存储最后计算的快照，如果存储中的数据没有改变，则返回与上次相同的快照。如何确定可变数据是否已更改取决于您的可变存储。

### My `subscribe` function gets called after every re-render

这个 `subscribe` 函数是在组件内部定义的，所以它在每次重新渲染时都是不同的：

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // 🚩 Always a different function, so React will resubscribe on every re-render
  function subscribe() {
    // ...
  }

  // ...
}
```

如果您在重新渲染之间传递不同的 `subscribe` 函数，React 将重新订阅您的 `store`。如果这会导致性能问题并且您希望避免重新订阅，请将 `subscribe` 函数移到外面：

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

// ✅ Always the same function, so React won't need to resubscribe
function subscribe() {
  // ...
}
```

或者，将 `subscribe` 包装到 `useCallback` 中以仅在某些参数更改时重新订阅：

```jsx
function ChatIndicator({ userId }) {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // ✅ Same function as long as userId doesn't change
  const subscribe = useCallback(() => {
    // ...
  }, [userId]);

  // ...
}
```



