# HTTP-API 与 React 生命周期方法和挂钩的集成

> 原文：<https://www.moesif.com/blog/technical/HTTP-API-Integration-with-React-Lifecycle-Methods-and-Hooks/>

## 为什么

当我们创建单页面应用程序(SPA)时，我们经常需要集成 API。有时是第三方 API，但至少是我们自己的后端来获取我们需要显示的数据。这些 API 基于 HTTP 或 WebSocket 协议，每种协议都有自己的连接建立和拆除要求。

在本文中，我解释了 HTTP APIs 的基本集成。

## 什么

HTTP 是一种无状态协议。这是从服务器获取数据的最简单的方法。

*   调用[获取函数](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
*   得到一个[承诺](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
*   等到承诺解决
*   用新数据更新应用程序

有时 HTTP 请求会失败，有时我们会取消请求，因为数据还没有到达，但已经不再需要了。

### 生命周期方法

[生命周期方法](https://reactjs.org/docs/state-and-lifecycle.html)是具有特殊名称的组件方法，由 React 对特定事件进行调用。

例如，`componentDidMount`将在 React 渲染一个组件到 DOM 后被调用。

### 钩住

钩子是 React 的一个新的部分，它允许我们做我们用生命周期方法做的事情，但是不需要创建一个组件类，它们只和功能组件一起工作。

例如，每次 React 渲染一个组件时，都会调用提供给`useEffect`钩子函数的回调。

## 怎么

首先，让我们通过生命周期方法进行集成。

## 生命周期方法

为了使用生命周期方法，我们需要创建一个具有三种方法的类组件，`render`、`componentDidMount`和`componentWillUnmount`。

```py
class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: true,
      data: null,
      error: null
    };
  }

  async componentDidMount() {
    this.abortController = new AbortController();

    try {
      const response = await fetch(API_URL, {
        signal: this.abortController.signal
      });

      if (response.status >= 300)
        throw new Error(response.statusText);

      const data = await response.json();

      this.setState({ loading: false, data });
    } catch (e) {
      if (e.name != "AbortError") this.setState({ error: e.message });
    }
  }

  componentWillUnmount() {
    this.abortController.abort();
  }

  render() {
    const { data, error, loading } = this.state;

    if (!!error) return <h2>{error}</h2>; 
    if (loading) return <h2>Loading...</h2>; 
    return <h2>{data}</h2>;
  }
} 
```

让我们一步一步来。

**1 -定义`constructor`中的状态**

对于 HTTP 请求，我们需要三种状态。`loading`、`data`和`error`。

**2 -在`componentDidMount`生命周期方法**中启动请求

我们在这里使用异步函数，所以我们可以用`await`处理`fetch`函数的承诺。

首先，我们需要定义一个允许我们取消 HTTP 请求的 [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController#Examples) 。然后我们称`try`块中的`fetch`为`await`块中的`response`。

我们还将`abortController`的`signal`传递给 fetch 调用，用请求连接控制器。当我们调用`abortController`的`abort`方法时，这个信号用来取消请求。

如果我们请求的`status`不是错误代码，我们假设数据已经准备好被解析；我们将它添加到我们的状态，并将`loading`设置为`false`。

如果`fetch`调用抛出一个错误，我们从服务器得到一个错误代码，或者`abortController`的`abort`方法被调用，我们`catch`这个错误并呈现一个错误。

**3 -在`componentWillUnmout`生命周期方法**中取消请求

因为我们保存了对`abortController`到`this`的引用，所以我们可以在`componentWillUnmount`方法中使用它。React 在组件从 DOM 中移除之前调用这个方法。

对`abort`的调用导致对`fetch`承诺的拒绝。

在`catch`块中，如果错误不是`AbortError`，我们只调用`setState`方法，因为我们知道 React 会从 DOM 中删除我们的组件。

**4 -渲染不同的状态**

最后，我们必须渲染不同的状态。主要的逻辑在生命周期方法中，所以`render`方法不再需要太多的逻辑。

### 钩住

要使用钩子，我们必须创建一个功能组件。在这个函数中，我们必须使用两个钩子，`useState`用来存储我们的状态，`useEffect`用来处理 API 调用。

```py
function MyComponent() {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);

  useEffect(async () => {
    const abortController = new AbortController();

    try {
      const response = await fetch(API_URL, {
        signal: abortController.signal
      });

      if (response.status >= 300)
        throw new Error(response.statusText);

      const data = await response.json();

      setData(data);
      setLoading(false);
    } catch (e) {
      if (e.name != "AbortError") setError(e.message);
    }

    return () => abortController.abort();
  }, []);

  if (!!error) return <h2>{error}</h2>; 
  if (loading) return <h2>Loading...</h2>; 
  return <h2>{data}</h2>; } 
```

**1 -首先用`useState`挂钩**设置状态

`useState`钩子接受一个初始值，并返回一个新的状态变量和一个 setter 函数。每次调用 setter 时，它将导致 React 使用状态变量中的新值重新呈现组件。

**2 -用`useEffect`钩子**开始请求

`useEffect`钩子接受一个回调，每当 React 呈现组件时(例如，当我们调用 setter 函数时)调用这个回调。

当我们将一个空数组作为第二个参数传递给`useEffect`时，回调只在第一次渲染后执行。这允许我们模拟`componentDidMount`生命周期方法的行为。

回调中的逻辑与生命周期方法示例中的逻辑基本相同。主要的区别是缺少了`this`，因为我们没有类组件，并且我们使用了`useState`钩子的设置器。

**3 -用返回的函数**取消请求

我们从提供给`useEffect`钩子的回调中返回的函数在组件从 DOM 中移除之前被执行。这允许我们模拟`componentWillUnmout`生命周期方法的行为。

我们调用`abortController`的`abort`方法并完成。

**4 -渲染不同的状态**

为了渲染，我们可以使用由`useState`钩子返回的状态变量。大部分逻辑都在我们传递给`useEffect`的回调函数中，所以这里没什么要做的。

## API 分析和监控

顺便说一句，如果你对如何将 [API 分析](https://www.moesif.com/features/api-analytics)添加到你的 React SPA 中感到好奇，看看[这个例子](https://github.com/Moesif/moesif-react-boilerplate-example)。

## 结论

将 API 调用集成到 React 组件中的两种方式主要取决于个人喜好。有些人更喜欢面向对象的方法；其他人希望更实用。React 允许您选择任何一种方式，都允许错误处理和取消。