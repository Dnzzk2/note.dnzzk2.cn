---
description: React中的TypeScript类型
head:
  - - meta
    - name: keywords
      content: TypeScript,React,React中的ts。
  - - link
    - rel: canonical
      content: https://note.dnzzk2.icu/framework/React/type-in-react
  - - meta
    - property: og:title
      content: React中的TypeScript类型
  - - meta
    - property: og:description
      content: 了解React中的TS类型，学会如何声明类型。
  - - meta
    - property: og:url
      content: https://note.dnzzk2.icu/framework/React/type-in-react
---

# React 中的 TypeScript 类型

## 描述 JSX 的类型

使用 `React.ReactElement` 描述 JSX 的类型，如果传过来的不一定是组件也可能是 `number` 或是 `null` ，则使用 `React.ReactNode` 。

`ReactNode` 包含 `ReactElement`、或者 `number`、`string`、`null`、`boolean` 等可以写在 JSX 里的类型。

```ts
type ReactNode =
  | ReactElement
  | string
  | number
  | Iterable<ReactNode>
  | ReactPortal
  | boolean
  | null
  | undefined
  | DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES[
    keyof DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES];
```

`ReactNode` > `ReactElement` > `JSX.Element`

一般情况下，描述一个 JSX 类型，使用 `React.ReactNode`

## 函数组件的类型

`React.FunctionComponent` = `FC`

## hooks 的类型

### useState

推导类型或者`useState<Type>()`

### useRef

- 保存 DOM 引用，参数传 `null`，ref.current 只读

```tsx
const ref = useRef<HTMLDivElement>(null);
```

- 数据，参数不传 `null`

```tsx
const ref = useRef<{ num: number }>({ num: 1 });
```

### useImperativeHandle

```tsx
import { useRef } from "react";
import { useEffect } from "react";
import React from "react";
import { useImperativeHandle } from "react";

interface GuangProps {
  name: string;
}

interface GuangRef {
  aaa: () => void;
}

const Guang: React.ForwardRefRenderFunction<GuangRef, GuangProps> = (
  props,
  ref
) => {
  const inputRef = useRef<HTMLInputElement>(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        aaa() {
          inputRef.current?.focus();
        },
      };
    },
    [inputRef]
  );

  return (
    <div>
      <input ref={inputRef}></input>
      <div>{props.name}</div>
    </div>
  );
};

const WrapedGuang = React.forwardRef(Guang);

function App() {
  const ref = useRef<GuangRef>(null);

  useEffect(() => {
    console.log("ref", ref.current);
    ref.current?.aaa();
  }, []);

  return (
    <div className="App">
      <WrapedGuang name="guang" ref={ref} />
    </div>
  );
}

export default App;
```

`React.ForwardRefRenderFunction<ref,props>`

`React.forwardRef<ref,props>`

第一个参数是 ref 的内容类型，第二个是 props 的类型

`useImperativeHanlde` 可以有两个类型参数，一个是 ref 内容的类型，一个是 ref 内容扩展后的类型。

```tsx
useImperativeHandle < GuangRef,
  { bbb: string } &&
    GuangRef >
      (ref,
      () => {
        return {
          aaa() {
            inputRef.current?.focus();
          },
          bbb: "bbb",
        };
      },
      [inputRef]);
```

::: tip
如有疑惑，请跳转至 [useRef — React 中常见的 Hooks](../React/hooks.md#ref-从子组件传到父组件) 查看基础知识，理解 `forward`。
:::

### useReducer

`useReducer<Reducer<Data,Action>,string>>`，可以传一个参数类型也可以传两个参数类型

传一个参数类型的时候，`Reducer<Data,Action>`，Data 是 data 的类型，Action 是 action 的类型

传两个参数的时候，`useReducer<Reducer<Data,Action>,string>>(recuder,'1',(params)=>{return {}})`，第二个参数就是初始化函数的参数的类型

### useCallback

`useCallback` 的类型参数是传入的函数的类型：

```tsx
const fun = useCallback<() => number>(() => {
  return 1;
}, []);
```

### useMemo

`useMemo` 的类型参数是传入的函数的返回值类型：

```tsx
const fun = useMemo<{ a: number }>(() => {
  return { a: 1 };
}, []);
```

### useContext

`useContext` 的类型参数是 Context 内容的类型：

```tsx
const context = createContext(1);

const num = useContext<number>(context);
```

## 参数类型

### PropsWithChildren

```tsx
interface Props {
  content: React.ReactNode;
  children:React.ReactNode;
}

function A(props:Props)=>{
  return <div>{props.content}{props.children}</div>
}

const B = ()=>{
  return <A content={<div>123</div>}>
            <div>children</div>
         </div>
}

```

children 类型的定义其实不需要自己写，可以使用`PropsWithChildren`。

```tsx
type Props = PropsWithChildren<{content:React.ReactNode}>

function A(props:Props)=>{
  return <div>{props.content}{props.children}</div>
}

const B = ()=>{
  return <A content={<div>123</div>}>
            <div>children</div>
         </div>
}

```

`PropsWithChildren` 的定义。

```ts
type PropsWithChildren<P = unKnown> = P & { children?: ReactNode | undefined };
```

### CSSProperties

通过 props 传递 css。

```tsx
type Props = PropsWithChildren<{
  content:React.ReactNode,
  color:CSSProperties['color'],
  styles?:CSSProperties
  }>

function A(props:Props)=>{
  return <div>{props.content}{props.children}</div>
}

const B = ()=>{
  return <A content={<div>123</div>} color="#7F2323" styles={{}}>
            <div>children</div>
         </A>
}
```

### HTMLAttributes

通过继承 `HTMLAttributes` ，使组件获得 HTML 普通标签的提示。

::: tip
`Attributes` 是属性的意思。
:::

```tsx
import React, { HTMLAttributes } from "react";

interface Props extends HTMLAttributes<HTMLDivElement> {}

function A(props: Props) {
  return <div>a</div>;
}

function App() {
  return (
    <div>
      <A prefix="">
        <button>7777</button>
      </A>
    </div>
  );
}

export default App;
```

`HTMLAttributes` 中的 `HTMLDivElement` 参数，是 onClick、onMouseMove 等事件处理函数的类型参数。

继承 `HTMLAttributes` ，只会有通用 HTML 属性，某些属性是某些标签特有的，我们可以指定他们的属性，如 `FormHTMLAttributes`、`AnchorHTMLAttributes` 等。

下面是 `a` 标签特有的`href` 属性。

```ts
interface AnchorHTMLAttributes<T> extends HTMLAttributes<T> {
  download?: any;
  href?: string | undefined;
  hrefLang?:string | undefined;
  media?: string undefined;
  ping?: string |undefined;
  target?:HTMLAttributeAnchorTarget undefined;
  type?: string |undefined;
  referrerPolicy?:HTMLAttributeReferrerPolicy | undefined;
}

```

### ComponentProps

`ComponentProps`，在类型参数中填写标签名，可以获取该标签的类型。

```tsx
interface Props extends ComponentProps<"a"> {}
```

### EventHandler

组件中传递处理函数，如 handlerClick。

```tsx
interface Props {
  handlerClick: MouseEventHandler;
}

const A = (props: Props) => {
  return <div onClick={props.handlerClick}>123</div>;
};

const App = () => {
  return (
    <A
      handlerClick={(e) => {
        console.log(e);
      }}
    />
  );
};
```

这种参数就要用 xxxEventHandler 的类型，比如 `MouseEventHandler`、`ChangeEventHandler` 等，它的类型参数是元素的类型。

不使用参数，也可以自己声明一个函数

```tsx
interface Props {
  handlerClick: (e: MouseEvent<HTMLDivElement>) => void;
}

const A = (props: Props) => {
  return <div onClick={props.handlerClick}>123</div>;
};

const App = () => {
  return (
    <A
      handlerClick={(e) => {
        console.log(e);
      }}
    />
  );
};
```
