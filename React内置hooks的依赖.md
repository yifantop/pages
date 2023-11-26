# React内置hooks的依赖

涉及到dependencies这个参数的React hooks有useEffect、useCallback、useMemo

useEffect、useCallback、useMemo都可传入两个参数，第一个参数都是函数，在useEffect中它叫setup，在useCallback中它叫fn，而在useMemo中叫calculateValue，只是名称不一样。第二个参数是dependencies（可选），这篇文章主要讨论第二个参数，它是一个像[dep1, dep2, dep3]一样的内联数组，原则上，上述hooks第一个参数的函数体中只要用到了reactive values，都应该被写进dependencies数组中，reactive values包括props、state、在组件body中声明的全部variables和functions。

下面讨论每一种reactive values和hooks的关系。

对于props和state，它们在变化时都会触发组件重渲染，重渲染过程中执行到useEffect、useCallback、useMemo，都会用Object.is()逐一比对每一个dependency，如果有变化就重新执行。

对于组件中声明的variables和functions，需要防止每一次重渲染它们都被赋予新值，造成hooks不必要的执行，下面是将variables和functions作为dependencies时出现的问题以及解决方法。

问题一：

```jsx
function App() {
  // 每一次重渲染，person都是新对象
	const person = {
    name: 'xxx'
  };

  useEffect(() => {
    // ...
  }, [person])
}
```

解决方法就是将对象中的属性单独抽取出来，让hooks依赖这些属性，或者将person用useMemo封装。

```jsx
function App() {
	const person = {
    name: 'xxx'
  };
  const { name } = person;

  useEffect(() => {
    // ...
  }, [name])
}
```

问题二：对于functions，每一次的重渲染都会声明新的函数，即使它的功能没有任何变化。

```jsx
function App() {
  // 每一次重渲染，fn都是新的函数
	const fn = () => {
    // ...
  }

  useEffect(() => {
    // ...
  }, [fn])
}
```

解决方法：可以将它用useCallback封装。

```jsx
function App() {
	const fn = useCallback(() => {
    // ...
  }, []);

  useEffect(() => {
    // ...
  }, [fn])
}
```

除了以上案例，再讨论一下useRef，不推荐将useRef返回的结果作为以上hooks的dependency，不管是ref还是ref.current，第一个原因是即使组件重渲染，ref都是相同的，把ref作为依赖没有意义；第二个原因是ref.current的变化并不会引发重渲染，即使它确实改变了，而将来如果组件因其它原因重渲染，useEffect、useCallback、useMemo就可能出现意外执行的情况，让组件渲染结果不可控制。

如果一定要依赖ref，可以采用以下的写法，将current中的属性解构出来，再作为依赖。不仅是ref，对于组件内部对象也推荐用这种方式。

```jsx
function App() {
	const ref = useRef(null);
  if (!ref.current) {
    ref.current = {
      name: 'xxx'
    };
  }
  const { name } = ref.current;
  
  useEffect(() => {
    // ...
  }, [name])
}
```

最后讨论外部对象，和ref一样，外部对象的属性变化也不会触发使用这个属性的组件重渲染，因此，也不推荐将外部对象的属性作为useEffect、useCallback、useMemo的依赖。