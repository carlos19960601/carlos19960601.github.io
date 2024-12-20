---
title: 'React Rerender-Children as Props模式'
date: 2024-12-14T11:45:00+08:00
draft: false
tags: 
  - React
categories:
  - React
---

现在需要创建一个Block组件，需要根据scroll value来控制其显示的位置。

```jsx
const MainScrollableArea = () => {
  const [position, setPosition] = useState(300);
  const onScroll = (e) => {
    // calculate position based on the scrolled value 
    const calculated = getPosition(e.target.scrollTop); 
    // save it to state
    setPosition(calculated);
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      {/* pass position value to the new movable component */}
      <MovingBlock position={position} />
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  );
```

根据 [React Rerender-Move State Down模式](./React%20Rerender-Move%20State%20Down模式.md)我们了解到这会导致整个`MainScrollableArea`都会重新渲染。

于是我们可以将不依赖于`position`的组件提取出来。

```jsx
const ScrollableWithMovingBlock = ({children}) => {
  const [position, setPosition] = useState(300);

  const onScroll = (e) => {
    const calculated = getPosition(e.target.scrollTop); 
    setPosition(calculated);
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {/* slow bunch of stuff used to be here, but not anymore */}
      {children}
    </div>
  );
};
```

```jsx
const App = () => { 
  return (
    <ScrollableWithMovingBlock>
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </ScrollableWithMovingBlock>
  );
};
```

这样`VerySlowComponent`、`BunchOfStuff`、`OtherStuffAlsoComplicated`这些组件就不会重新渲染了。有人可能会疑惑，为什么这样就不会重新渲染了呢？`VerySlowComponent`、`BunchOfStuff`、`OtherStuffAlsoComplicated`这些组件仍然在`ScrollableWithMovingBlock`中，`position`发生改变了，为啥它们不会重新渲染呢？

这就需要了解Rerender的机制了。Rerender的时候，React会重新执行创建组件的函数，然后通过Object.is来判断是否有变化来判断是否重新创建。以上面的`ScrollableWithMovingBlock`为例，看看Rerender的过程。

```jsx
const ScrollableWithMovingBlock = ({children}) => {
  const [position, setPosition] = useState(300);

  const onScroll = (e) => {
    const calculated = getPosition(e.target.scrollTop); 
    setPosition(calculated);
  };

  return (
    <div className="scrollable-block" onScroll={onScroll}>
      <MovingBlock position={position} />
      {/* slow bunch of stuff used to be here, but not anymore */}
      {children}
    </div>
  );
};
```

当`position`发生变化的时候，会重新执行`ScrollableWithMovingBlock`函数，判断其返回值中所有的Object是否有变化。`MovingBlock`是在本地创建的，所以每次都是新创建的，`Object.is`判断为false。所有`MovingBlock`会重新创建。对于`children`由于是外部创建的，所以`Object.is`判断为true，children不会重新渲染。