+++
title = '关于React的rerender'
date = 2024-12-04T16:06:31+08:00
draft = false
+++

作为一个React初学者，在写React程序的时候经常遇到rerender的问题。由于写的都比较简单，也没有关注rerender带来的问题。但是遇到问题了又到处Google找问题，还是比较浪费时间的。所以这次下定决心学习一下React的Rerender的机制。

当Component第一次出现在屏幕上，我们称之为`mounting`。React会初始化Component的state, 运行hooks，添加element到dom中。

当React检测到Component不再需要时，我们称之为`unmounting`。React会执行clean-up，销毁组件及其相关的state，最后从dom中移除element。

`rerender`是在已经存在的Component的基础上，一些state发生了变化，导致Component重新渲染。相比于`mounting`会更加轻量。

每次rerender都始于state的改变。你可以把React App想象成1棵树。state变化的那个节点会导致这个节点下面的所有分支节点都重新渲染。

![](/关于React的rerender/rerender-state.png)

你可能听说过props改变了也会导致Component rerender。但这是个误解，出现这个根本原因还是因为state发生了变化。来看下面这个例子

```jsx  
const App = () => {
  // local variable won't work 
  let isOpen = false;
  return (
    <div className="layout">
      {/* nothing will happen */}
      <Button onClick={() => (isOpen = true)}>
        Open dialog
      </Button>
      {/* will never show up */} 
      {isOpen ? (
        <ModalDialog onClose={() => (isOpen = false)} /> 
      ) : null}
    </div> 
  );
};
```

当点击Button的时候，isOpen会被改变，但是ModalDialog并没有重新渲染。local variable不会被React追踪，所以ModalDialog不会重新渲染。 


再看看下面的代码。isOpen的值发生改变，会导致整个App都会重新渲染。`VerySlowComponent`、`BunchOfStuff`、`OtherStuffAlsoComplicated`这些组件并不依赖isOpen，但是都进行了rerender。

```jsx
const App = () => {
  // our state is declared here
  const [isOpen, setIsOpen] = useState(false);
  return (
    <div className="layout">
      {/* state is used here */}
      <Button onClick={() => setIsOpen(true)}>
        Open dialog
      </Button>
      {/* state is used here */} 
      {isOpen ? (
        <ModalDialog onClose={() => setIsOpen(false)} /> 
      ) : null}
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  ); 
};
```

一种方式是`React.memo`进行包装。`React.memo`会对props进行浅比较，如果props没有改变，就不会重新渲染。

![](/关于React的rerender/react-memo.png)

但是这个例子中，并不需要使用memo，有更好的方式就是将state下沉下去。

```jsx
const ButtonWithModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);
    // render only Button and ModalDialog here
  return ( 
    <>
      <Button onClick={() => setIsOpen(true)}> 
        Open dialog
      </Button>
      {isOpen ? (
        <ModalDialog onClose={() => setIsOpen(false)} /> 
      ) : null}
    </> 
  );
};
```      

```jsx
const App = () => {
  return (
    <div className="layout">
      {/* here it goes, component with the state inside */} 
      <ButtonWithModalDialog />
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  ); 
};
```

现在点击Button的时候，只会重新渲染ButtonWithModalDialog，不会重新渲染其他组件。

![](/关于React的rerender/move-state-down.png)


**自定义hook的注意事项**

在上面这个例子中，可以简单的定义一个自定义hook -> `useModalDialog`

```jsx
const useModalDialog = () => {
  const [isOpen, setIsOpen] = useState(false);

  return {
    isOpen,
    open: () => setIsOpen(true), 
    close: () => setIsOpen(false),
  }; 
};
```

```jsx
const App = () => {
  // state is in the hook now
  const { isOpen, open, close } = useModalDialog();
  return (
    <div className="layout">
      {/* just use "open" method from the hook */} 
      <Button onClick={open}>Open dialog</Button>
      {/* just use "close" method from the hook */}
      {isOpen ? <ModalDialog onClose={close} /> : null}
      <VerySlowComponent />
      <BunchOfStuff />
      <OtherStuffAlsoComplicated />
    </div>
  ); 
};
```

自定义hook隐藏了在App中有state。所以当state发生变化会导致整个App重新渲染。尽管不在App中使用，或者自定义hook没有返回什么，也会导致整个App重新渲染。比如下面这样。

```jsx
const useModalDialog = () => {
  const [width, setWidth] = useState(0);
  useEffect(() => {
    const listener = () => {
      setWidth(window.innerWidth); 
    }
    
    window.addEventListener('resize', listener);
    
    return () => window.removeEventListener('resize', listener);
  }, []);
    // return is the same
  return ...
}
```

整个App都会在resize的时候重新渲染。即使这个width没有返回。

Hooks就像裤子上面的口袋，你手举10kg的哑铃，将哑铃放在口袋里面并不能改变你携带10kg重物的事实。你需要将哑铃放置在手推车里面。

即使hook依赖另外一个hook，也会导致整个App重新渲染。

```jsx
const useResizeDetector = () => {
  const [width, setWidth] = useState(0);
  useEffect(() => {
    const listener = () => {
      setWidth(window.innerWidth); 
    };
    
    window.addEventListener('resize', listener);
    
    return () => window.removeEventListener('resize', listener);
  }, []);
  return null;
}

const useModalDialog = () => {
  // I don't even use it, just call it here 
  useResizeDetector();
  // return is the same
  return ...
}

const App = () => {
  // this hook uses useResizeDetector underneath that triggers
  // state update on resize
  // the entire App will re-render on every resize! 
  const { isOpen, open, close } = useModalDialog();
  return // same return 
}
```

为了解决这个问题，还是需要将state下沉下去。

```jsx
const ButtonWithModalDialog = () => {
  const { isOpen, open, close } = useModalDialog();
    // render only Button and ModalDialog here
  return ( 
    <>
      <Button onClick={open}>Open dialog</Button>
      {isOpen ? <ModalDialog onClose={close} /> : null} 
    </>
  ); 
};
```

### 参考资料

* [React re-render 完全指南](https://juejin.cn/post/7254443448562974775)
* [Advanced React deep dives, investigations, performance patterns and techniques (Nadia Makarevich).pdf](/books/Advanced%20React%20deep%20dives,%20investigations,%20performance%20patterns%20and%20techniques%20(Nadia%20Makarevich)%20(Z-Library).pdf)