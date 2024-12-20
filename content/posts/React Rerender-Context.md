---
title: 'React Rerender Context'
date: 2024-12-18T17:56:13+08:00
draft: false
tags:
  - React
categories:
  - React
---

### Context的作用

![](/关于React的rerender/react-context.png)

Context可以让我们在组件树中传递数据，而不需要通过props一层层传递。

但是有一些需要注意的点。

* Context consumers will re-render when the value on the Provider changes.
* All of them will re-render, even if they don't use the part of the value that actually changed.
* Those re-renders can't be prevented with memoization (easily).


### Context的value改变

```jsx
const NavigationController = ({ children }) => {
  const [isNavExpanded, setIsNavExpanded] = useState();
  const toggle = () => setIsNavExpanded(!isNavExpanded);

  const value = { isNavExpanded, toggle };

  return (
    <Context.Provider value={value}>
        {children}
    </Context.Provider>
  ); 
};

const useNavigation = () => useContext(Context);
```

每次value改变的时候， 使用`useNavigation`的组件都会重新渲染。这很正常，但是如果是因为其他原因，导致value改变呢？比如


```jsx
const Layout = ({ children }) => {
  const [scroll, setScroll] = useState();
  useEffect(() => {
    window.addEventListener('scroll', () => { setScroll(window.scrollY);}); 
  },[]);

  return (
    <NavigationController>
      <div className="layout">{children}</div>
    </NavigationController>
  ); 
};
```

现在，只要window滚动，就会造成`NavigationController`重新渲染, `NavigationController`重新渲染导致`value`重新创建，导致依赖`value`的组件重新渲染。

其实`value`里面的`isNavExpanded`和`toggle`并没有发生变化，为了解决这个问题，可以使用`useMemo`来缓存`value`。

```jsx
const NavigationController = ({ children }) => {
  const [isNavExpanded, setIsNavExpanded] = useState();

  const toggle = useCallback(() => { 
    setIsNavExpanded(!isNavExpanded);
  }, [isNavExpanded]);
  
  const value = useMemo(() => { 
    return { isNavExpanded, toggle };
  }, [isNavExpanded, toggle]);
  
  return (
    <Context.Provider value={value}>
      {children}
    </Context.Provider>
  ); 
};
```


接下来再来看一个场景，代码如下：

```jsx
import React, { ReactNode, useCallback, useContext, useEffect, useMemo, useState } from 'react';

import { AnotherVerySlowComponent, VerySlowComponent } from './components/very-slow-component';
import './styles.scss';

const Context = React.createContext({
  isNavExpanded: false,
  toggle: () => {},
  close: () => {},
  open: () => {},
});

const NavigationController = ({ children }: { children: ReactNode }) => {
  const [isNavExpanded, setIsNavExpanded] = useState(false);

  const toggle = useCallback(() => {
    setIsNavExpanded(!isNavExpanded);
  }, [isNavExpanded]);

  const open = useCallback(() => {
    setIsNavExpanded(true);
  }, []);
  const close = useCallback(() => {
    setIsNavExpanded(false);
  }, []);
  const value = useMemo(() => {
    return { isNavExpanded, toggle, close, open };
  }, [isNavExpanded, toggle, close, open]);

  return <Context.Provider value={value}>{children}</Context.Provider>;
};

const useNavigation = () => useContext(Context);

const AdjustableColumnsBlock = () => {
  const { isNavExpanded } = useNavigation();
  return isNavExpanded ? <div>two block items here</div> : <div>three block items here</div>;
};

const withNavigationOpen = (AnyComponent: any) => {
  // wrap the component from the arguments in React.memo here
  const AnyComponentMemo = React.memo(AnyComponent);

  return (props: any) => {
    const { open } = useContext(Context);

    // return memoized component here
    // now it won't re-render because of Context changes
    // make sure that whatever is passed as props here don't change between re-renders!
    return <AnyComponentMemo {...props} openNav={open} />;
  };
};

const MainPart = withNavigationOpen(({ openNav }: { openNav: () => void }) => {
  useEffect(() => {
    // won't be triggered when context value changes
    // because toggleNav is coming from memoized HOC
    console.info('Main part re-render');
  });

  return (
    <>
      <div>
        <button onClick={openNav}>click to expand nav - inside heavy component</button>
      </div>
      <VerySlowComponent />
      <AnotherVerySlowComponent />
      <AdjustableColumnsBlock />
    </>
  );
});

const ExpandButton = () => {
  const { isNavExpanded, toggle } = useNavigation();

  useEffect(() => {
    console.info('Button that uses Context re-renders');
  });

  return <button onClick={toggle}>{isNavExpanded ? 'collapse <' : 'expand >'}</button>;
};

const SidebarLayout = ({ children }: { children: ReactNode }) => {
  const { isNavExpanded } = useNavigation();
  return (
    <div className="left" style={{ flexBasis: isNavExpanded ? '50%' : '20%' }}>
      {children}
    </div>
  );
};

const Sidebar = () => {
  return (
    <SidebarLayout>
      {/* this one will control the expand/collapse */}
      <ExpandButton />

      <ul>
        <li>
          <a href="#">some links</a>
        </li>
      </ul>
    </SidebarLayout>
  );
};

const Layout = ({ children }: { children: ReactNode }) => {
  const [scroll, setScroll] = useState(0);

  useEffect(() => {
    window.addEventListener('scroll', () => {
      setScroll(window.scrollY);
    });
  }, []);

  return (
    <NavigationController>
      <div className="three-layout">{children}</div>
    </NavigationController>
  );
};

const Page = () => {
  return (
    <Layout>
      <Sidebar />
      <MainPart />
    </Layout>
  );
};

export default function App() {
  return (
    <>
      <h3>Very slow "Page" component - click on expand/collapse to toggle nav</h3>
      <p>Scrolling causes re-render of everything that uses Context</p>
      <Page />
    </>
  );
}
```

上面的代码实现了一个`Sidebar`和`MainPart`2个部分，`Sidebar`中有个button可以控制折叠，`MainPart`中根据`Sidebar`是否折叠展示不同的layout。

`MainPart`中有些slow的组件。在`withNavigationOpen`中使用了meme,如果没有meme，每次`MainPart`都是在`Sidebar`折叠的时候重新渲染。但是`MainPart`中的slow的组件并不依赖于`Sidebar`是否折叠。

造成这个问题的原因是`withNavigationOpen`中使用了`useContext(Context)`, 由于`isNavExpanded`改变，导致`Context`的value变化，`AnyComponent`就会重新渲染，但是`open`其实是不依赖于`isNavExpanded`，所以`AnyComponent`加上memo后，props和openNav没有变化，就不会重新渲染了。

```jsx
const withNavigationOpen = (AnyComponent: any) => {
  // wrap the component from the arguments in React.memo here
  const AnyComponentMemo = React.memo(AnyComponent);

  return (props: any) => {
    const { open } = useContext(Context);

    // return memoized component here
    // now it won't re-render because of Context changes
    // make sure that whatever is passed as props here don't change between re-renders!
    return <AnyComponentMemo {...props} openNav={open} />;
  };
};
```