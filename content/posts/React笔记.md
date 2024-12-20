---
title: 'React笔记'
date: 2024-08-28T14:43:33+08:00
draft: false
categories:
  - React
---

## Twitter/X 博主[@_georgemoller](https://x.com/_georgemoller)的分享

主要记录Twitter博主[@_georgemoller](https://x.com/_georgemoller)分享的React技巧。对于我这种React小白很使用。于是搬运过来，并附上个人的一些理解。如有问题请邮箱本人，欢迎各位指正。

### Solve re-render issues with composition

![](/React笔记/georgemoller_1c2c3b.gif)

> 个人理解：功能单一，利用组合模式减少耦合，带来不必要的重新渲染


### Utility Types For Enhanced Reusablity

![](/React笔记/georgemoller_9378eb.gif)

> 个人理解：Typescript Pick的使用


### Bad use case for useCallback

![](/React笔记/georgemoller_ed1930.gif)

> useCallback 使用在复杂逻辑的组件，简单组件不会消耗造成性能问题时不必使用

### Intersection types to extends native props in a component

![](/React笔记/georgemoller_29194f.gif)


### Don't listen for ref changes with useEffect

![](/React笔记/georgemoller_8216ce.gif)

> 本人曾经这么干过


### useEffect vs useLayoutEffect

![](/React笔记/georgemoller_d587ed.gif)

> 个人理解：useLayoutEffect在DOM更新后，浏览器绘制前执行。useEffect在DOM更新后，浏览器绘制后执行。

### Separation of Concerns in React

![](/React笔记/georgemoller_e8833e.gif)

> React中的关注点分离
> 这是一个重要的软件设计原则，在React开发中也同样适用。它指的是将代码按照不同的功能或责任进行划分和组织，使每个部分专注于特定的任务。在React中，这通常体现为:
> 1. 将UI和逻辑分离
> 2. 将状态管理与渲染逻辑分离
> 3. 将可复用的逻辑抽离成自定义Hook
> 4. 将大型组件拆分成更小的、功能单一的组件
> 
> 遵循这一原则可以提高代码的可维护性、可读性和可重用性。

### Easy to read abstraction over array state

![](/React笔记/georgemoller_ae4db6.gif)

> 个人理解：对数组状态进行抽象，简化逻辑

### How to import Server Components from Client Components

![](/React笔记/georgemoller_c72511.gif)

> 个人理解：服务端渲染框架中使用的小技巧

### Avoid Transforming Data in useEffect

![](/React笔记/georgemoller_bc91e5.gif)

> 个人理解：在useEffect中避免对数据进行转换，避免不必要的重新渲染
> 
> 原因：
>   * 性能问题：在 useEffect 中转换数据可能导致不必要的重渲染。
>   * 代码复杂性：可能使组件逻辑变得难以理解和维护。
> 
> 更好的做法：
>   * 在渲染过程中直接转换数据。
>   * 使用 useMemo 缓存计算结果。

### How and when to use flushSync

![](/React笔记/georgemoller_cf6586.gif)

### Defer creation of non-primitive values if you are using them within a dependency array

![](/React笔记/georgemoller_a5e9d8.gif)

> 个人理解: useEffect使用Tips

### Avoid premature optimization

![](/React笔记/georgemoller_622a12.gif)

### How to expose custom refs with useImperativeHandle hook

![](/React笔记/georgemoller_10ebf7.gif)


### Separate business logic from UI

![](/React笔记/georgemoller_e78ede.gif)

### How to virtualize large lists

![](/React笔记/georgemoller_2e4bec.gif)

### Avoid difficult to read conditionals in React

![](/React笔记/georgemoller_54b4e2.gif)


### Using inferred types in React

![](/React笔记/georgemoller_a5281a.gif)

### Avoid Provider Wrapping Hell

![](/React笔记/georgemoller_f90099.gif)


### useEffect cheatsheet

![](/React笔记/georgemoller_2aae18.gif)


## 开发中的学习

### useState vs useReducer 

个人理解useReducer是useState的进阶版本，useState适用于一些简单状态的管理。如果是多个简单状态之前存在复杂的逻辑关联，推荐使用useReducer去处理。

### @uidotdev/usehooks 中 useThrottle 和 useDebunce的区别是什么？

* 作用目的
  * useThrottle（节流）：
    目的是限制函数在一定时间内的执行次数。它确保函数在指定的时间间隔内最多被执行一次。
    
    例如，在滚动事件处理中，当用户快速滚动页面时，使用节流可以避免频繁地执行某个昂贵的操作，如发送网络请求或进行大量的计算，从而提高性能。

  * useDebounce（防抖）：
    主要用于在用户输入或触发某个事件后，等待一段时间，如果在这段时间内没有新的触发，则执行相应的函数。
    
    常见的应用场景是在搜索框输入时，当用户停止输入一段时间后才进行搜索请求，避免在用户输入过程中频繁发送请求。

* 执行时机
  * useThrottle：
    当触发事件开始时，函数会立即执行一次，然后在指定的时间间隔内，无论触发事件发生多少次，函数都不会再次执行。直到时间间隔结束后，下一次触发事件才能再次执行函数。
    
    例如，假设设置节流时间为 1 秒，当用户连续触发事件时，函数在第一秒内执行一次，之后的触发会被忽略，直到下一个 1 秒间隔开始。

  * useDebounce：
    当触发事件发生后，函数不会立即执行，而是启动一个定时器。如果在定时器到期之前又有新的触发事件，那么定时器会被重置。只有在定时器到期且没有新的触发事件时，函数才会执行。
    
    比如，设置防抖时间为 500 毫秒，当用户输入内容后，500 毫秒内如果没有新的输入，函数就会执行；如果在这 500 毫秒内又有新的输入，定时器会重新开始计时。

* 适用场景
  * useThrottle：
    适用于需要持续响应但又不能过于频繁执行的场景，比如滚动事件、窗口大小调整事件等，以减少资源消耗和提高性能。
    
    例如，在地图应用中，当用户拖动地图时，可以使用节流来限制地图更新的频率，避免过度的计算和网络请求。
  * useDebounce：
    非常适合处理用户输入、搜索框自动完成等场景，确保在用户停止操作一段时间后才执行相关操作，避免不必要的计算和网络请求。
    
    例如，在在线文档的自动保存功能中，可以使用防抖来避免频繁保存，而是在用户停止编辑一段时间后进行保存操作。

### 使用 ElementRef 类型定义ref的类型

记住`HTMLDivElement`是比较困难的

```jsx
const ref = useRef<HTMLDivElement>(null);
```

可以尝试

```jsx
const ref = useRef<ElementRef<"div">>(null);
```

## 从别人中的笔记学习

### React常用的设计模式

#### 组合模式

首先看一段代码，`Tabs`和`TabItem`是父子关系。

* `Tabs` 负责展示和控制对应的 `TabItem` 。绑定切换 `tab` 回调方法 `onChange`。当 `tab` 切换的时候，执行回调。
* `TabItem` 负责展示对应的 `tab` 项，向 `Tabs` 传递 `props` 相关信息。

```tsx
<Tabs onChange={ (type)=> console.log(type)  } >
    <TabItem name="react"  label="react" >React</TabItem>
    <TabItem name="vue" label="vue" >Vue</TabItem>
    <TabItem name="angular" label="angular"  >Angular</TabItem>
</Tabs>
```

**隐式混入 props** 

```tsx
function Item (props){
    console.log(props) 
    return <div> 名称： {props.name} </div>
}

function Groups (props){
    const newChilren = React.cloneElement(props.children, { author:'alien' })
    return  newChilren
}
```

通过`React.cloneElement`，向子组件中混入父组件的props。

**控制渲染**

```tsx
function Item (props){
    return <div> 名称： {props.name} </div>
}
/* Groups 组件 */
function Groups (props){
    const newChildren = []
    React.Children.forEach(props.children,(item)=>{
        const { type ,props } = item || {}
        if(isValidElement(item) && type === Item && props.isShow  ){
            newChildren.push(item)
        }
    })
    return  newChildren
}
```

通过遍历children，根据父组件的逻辑来控制子组件的渲染。

**内外层通信**

```tsx
function Item (props){
    return <div>
        名称：{props.name}
        <button onClick={()=> props.callback('let us learn React!')} >点击</button>
    </div>
}

function Groups (props){
    const handleCallback = (val) =>  console.log(' children 内容：',val )
    return <div>
        {React.cloneElement( props.children , { callback:handleCallback } )}
    </div>
}
```

向内层组件传递回调函数 callback ，内层通过调用 callback 来实现两层组合模式的通信关系。

#### render props模式

render props 模式和组合模式类似。区别不同的是，用函数的形式代替 children。函数的参数，由容器组件提供，这样的好处，将容器组件的状态，提升到当前外层组件中，这个是一个巧妙之处，也是和组合模式相比最大的区别。

```tsx
export default function App (){
    const aProps = {
        name:'《React进阶实践指南》'
    }
    return <Container>
        {(cProps) => <Children {...cProps} { ...aProps }  />}
    </Container>
}
```



参考资料: 
* [「React 进阶」 学好这些 React 设计模式，能让你的 React 项目飞起来](https://juejin.cn/post/7007214462813863950)