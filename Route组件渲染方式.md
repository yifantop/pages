# Route组件渲染

v5版本的react-router-dom中Route组件是最重要的组件，它的任务是当path匹配上URL后渲染UI，v5版本的Route组件有多种使用方式，本文讨论它们的渲染方式和区别。

页面上的`<Route />`在技术上都会被渲染，但如果path和URL并不匹配，它会被React按`null`来处理，也就是不挂载任何DOM（有一种情况例外，后文会提到）。只有path匹配上URL，才会真正渲染你传入的component。

要正确使用`<Route />`，通常我们会传入`path`和render methods，`path`不需过多解释。render methods有三种，分别是`component`、`render`、`children`，其实`<Route />`本就是React组件，三种render methods就是React组件三种属性，Route的这一套渲染方式、对属性的处理，和普通React组件相同。

我们使用`<Route />`渲染自己的React组件，通常想要获取到`Route props`，利用它们可以实现和路由相关的功能，`Route props`包括`match`、`location`、`history`，其中，`match`描述了路由匹配情况，`location`描述了App当前的路由位置，`history`提供了一套统一的跨平台API来控制页面进退。其实v5版本的react-router已经支持使用hooks来获取这三个`Route props`，它们分别是`useRouteMatch`、`useLocation`、`useHistory`，我们完全可以使用以下方式来获取到`match`、`location`、`history`，并且这也是官方推荐的写法。

```jsx
<Router>
  <Route path="/">
  	<Home />
  </Route>
</Router>

function Home() {
  const match = useRouteMatch();
  const location = usseLocation();
  const history = useHistory();
  
  return ...
}
```

但v5版本react-router依然保留了Route组件的`component`、`render`、`children`三个属性，是为了兼容React的类组件，类组件没有办法在内部使用hook，于是采用下面这种写法，react-router就会自动把三个`Route props`传给你的组件。

```jsx
<Route path="/" component={Home}>
```

`component`、`render`、`children`三种属性用起来是不同的，下面讨论它们的区别。

`component prop`，它接收一个React组件，可以是函数组件，也可以是类组件，但不要给它传入inline function，在`<Route />`内部，component prop只要改变，旧的组件实例将会被卸载，新的实例被创建，而父组件的每一次re-render，都会发现component prop是一个新的function，这导致每一次都不是在原组件实例上更新，而是卸载、重新挂载。

`render prop`，和`compnent prop`不同，我们为`render`传入一个function，`Route props`可以从这个function获取到。

```jsx
<Route path="/" render={routeProps => <Home {...routeProps} />}>
```

`children prop`，和`render prop`工作方式一样，区别是即使path没有匹配上URL，依然渲染UI，此时`match prop`是`null`，我们可以利用这种特性，根据是否匹配来动态调整UI，或者恒定执行一些动画。

```jsx
<Route
  path="/"
  children={({ match, ...rest }) => (
    {/* Animate will always render, so you can use lifecycles
        to animate its child in and out */}
    <Animate>
      {match && <Something {...rest}/>}
    </Animate>
  )}
/>
```

