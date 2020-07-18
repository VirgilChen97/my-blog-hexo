---
title: React Router 学习
toc: true
date: 2020-06-26 16:03:51
tags: React
categories: 找工作
---

# 简介

引用中文文档中的介绍：

> React Router 是一个基于React 之上的强大路由库，它可以让你向应用中快速地添加视图和数据流，同时保持页面与URL 间的同步。

也就是说使用了 React router，我们就可以通过URL来实现组件的展示与跳转。安装 React router 很简单，只需要：

```bash
npm install react-router-dom --save
```

<!-- more -->

# 快速上手

## 基本使用

首先我们使用 `create-react-app` 快速创建一个React单页应用，在你想要创建项目的目录执行：

```
npx create-react-app <项目名称>
```

首先创建一个 component 文件夹，在文件夹内创建 AppRouter.js，首先我们引入所有需要的包：

```js
import React from 'react'
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link
} from 'react-router-dom'
```

然后编写 AppRouter 组件本体：

```js
const AppRouter = () => {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/Users">Users</Link>
            </li>
          </ul>
        </nav>
        <Switch>
          <Route path="/about">
            <About />
          </Route>
          <Route path="/Users">
            <Users />
          </Route>
          <Route path="/">
            <Home />
          </Route>
        </Switch>
      </div>
    </Router>
  )
}
```

最后创建 `Home`，`About`，`User` 这三个组件：

```js
const About = () => {
  return <h2>About</h2>
}

const Home = () => {
  return <h2>Home</h2>
}

const Users = () => {
  return <h2>Users</h2>
}
```

最后别忘了：

```js
export default AppRouter
```

然后修改 App.js 删除除了import以外所有默认内容，在主页展示我们写好的 AppRouter组件：

```js
function App() {
  return (
    <AppRouter />
  );
}

export default App;
```

现在执行 `npm start`，观察运行效果。发现点击链接时浏览器的 URL 栏也会一起改变，但是实际上我们并没有加载任何新的 HTML 页面，这还是一个单页应用。

## Nested Routing

上一个例子展示了一个简单的路由，同样的我们也可以实现多级路由。对 AppRouter.js 进行修改，首先添加几个应用：

```js
import React from "react";
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link,
  useRouteMatch,
  useParams
} from "react-router-dom";
```

接着修改 AppRouter.js 下的 User：

```js
const Users = () => {
  // 通过 Hook 获取 match
  let match = useRouteMatch()
  let users = ['Alice', 'Bob','Jimmy']
  console.log(match)
  return (
    <div>
      <h2>Users</h2>
      <ul>
        {/*在User上循环输出链接*/}
        {users.map(user => {
          return (
          <li key={user} >
            <Link to={`${match.url}/${user}`}>{user}</Link>
          </li>
        )})}
      </ul>
      <Switch>
      <Route path={`${match.path}/:userId`}>
        <User />
      </Route>
      <Route path={`${match.path}`}>
        <h3>Please select a user</h3>
      </Route>
    </Switch>
    </div>
  )
}
```

然后添加一个新的User组件：

```js
const User = () => {
  // 通过 Hook 获取 路径变量
  let { userId } = useParams();
  return <h3>User name: {userId}</h3>
}
```

重新运行项目，选择不同的 user，浏览器路径也会跟随变化：

![](/img/(2020-06-26)React-Router-学习.md/2020-06-26-17-08-14.png)

# 开始

## URL 参数

在刚才的例子中我们已经接触到了 URL 参数，在 `Users` 组件中，`<Route>` 标签内的 path 不再是一个固定值，而是类似如下的格式：

```js
let match = useRouteMatch()
<Route path={`${match.path}/:userId`}>
  <User />
</Route>
```

在 js 中 "\`" 代表的是模板字符串，也就是说这里的 `path` 是 `match.path` 的值拼接上 `/:userId`。`match` 是什么现在先不管，这里 `/:userId` 就是一个 URL 参数。因此在这个 `<Route>` 标签内包裹的 `<User />` 组件便可以通过 `useParams()` 这个 hook 获取 `userId` 的值：

```js
let { userId } = useParams();
```

## 嵌套路由

在应用中 URL 肯定不止同一层级，例如可能会有一个管理用户的界面是 `/Users`，当需要查看或修改某一个用户时，界面可能是 `/Users/{userId}`。React Router很好的支持了嵌套路由。在之前的例子中，我们使用

```js
let match = useRouteMatch()
```

获取了一个 match 对象，match对象有两个很重要的属性 `path` 和 `url`。这两个属性告诉了我们当前组件的路径是什么。例如之前的例子，主 router 存在于 `AppRouter` 组件中，`User` 组件嵌套在 `AppRouter` 组件的一个 route 里，地址为 `/Users`。因此在 `User` 组件中我们使用 `useRouteMatch()` hook 获取 match 时，match 对象内的值为：

![](/img/(2020-06-26)React-Router-学习.md/2020-06-26-17-35-00.png)

因此在编写 `User` 组件内部的 route 时，我们只需要取出 `match` 中的 `path`，然后拼接上下一步的路径即可：

```js
<Route path={`${match.path}/:userId`}>
```

那么 `path` 和 `url` 有什么区别呢？在路径中全部为静态路径并且没有匹配规则时，两者的值是相同的。但是一旦包含变量或者匹配规则，`url` 显示的是现在具体的路径，而 `path` 包含的则当前的路径匹配规则。我们还是通过上面的例子来解释。在 `Users` 组件路由到 `User` 组件时，我们是这样写的：

```js
<Route path={`${match.path}/:userId`}>
    <User />
</Route>
```

现在我们修改 User，让其在终端输出 `match` 的值:

```js
const User = () => {
  // 通过 Hook 获取 路径变量
  let { userId } = useParams();
  let match = useRouteMatch();
  console.log(match)
  return <h3>User name: {userId}</h3>
}
```

![](/img/(2020-06-26)React-Router-学习.md/2020-06-26-17-43-47.png)

可以看到 `url` 的值是具体的 `"/Users/Alice"`，而 `path` 的值则是 `"/Users/:userId"`。

## 重定向

Web应用中重定向是一个很常见的操作，例如登录完成后跳转到首页，又或是未登录时自动跳转到登录页。在 react router 中有多种方式可以实现重定向。

1. 通过渲染 `redirect` 组件
2. 通过 `history`

我们通过例子来学习这两种方法，我们模拟一下登录认证的过程。在 component 下创建一个新的 AuthExample.js。首先我们引入需要的包：

```js
import React from "react";
import {
  BrowserRouter as Router,
  Switch,
  Route,
  Link,
  Redirect,
  useHistory,
  useLocation
} from "react-router-dom";
```

我们模拟的情况是一个网站中存在公共页面，也就是无需登录就可以访问的页面，同时还有私有页面，也就是必须登录才能访问的页面。如果在未登录的情况下访问私有页面，则会自动跳转到登录页。首先我们先编写一个模拟的认证功能：

```js
const fakeAuth = {
  isAuthenticated: false,
  // cb 为认证完成后的回调函数
  authenticate(cb) {
    fakeAuth.isAuthenticated = true;
    setTimeout(cb, 100); // fake async
  },
  signout(cb) {
    fakeAuth.isAuthenticated = false;
    setTimeout(cb, 100);
  }
};
```

接着我们编写页面的大概结构：

```js
const AuthExample = () => {
  return (
    <Router>
      <div>
        <ul>
          <li>
            <Link to="/public">Public Page</Link>
          </li>
          <li>
            <Link to="/protected">Protected Page</Link>
          </li>
        </ul>

        <Switch>
          <Route path="/public">
            <PublicPage />
          </Route>
          <Route path="/login">
            <LoginPage />
          </Route>
          {/*一个对<Route>组件的包装，见后文*/}
          <PrivateRoute path="/protected">
            <ProtectedPage />
          </PrivateRoute>
        </Switch>
      </div>
    </Router>
  );
}
```

你可能发现了 `PrivateRoute` 这个组件，他是我们对于普通 `Route` 组件的封装，其中包含了判断是否登录的逻辑：

```js
const PrivateRoute = ({ children, ...rest }) => {
  // Route 的 render 属性需要传入一个函数返回需要render的内容
  // 会自动传入 location 变量，即用户现在所在的路径
  const handleRender = ({location}) => {
    if(fakeAuth.isAuthenticated){
      // 如果登录了就渲染 childre
      return children
    }else{
      // 第一种方式：如果未登录就渲染 <Redirect> 进行重定向
      return (<Redirect
        to={{
          // 把现在的位置放到 /login 的state中，方便登陆完成后跳转回来
          pathname: "/login",
          state: { from: location }
        }}
      />)
    }
  }
  return (
    <Route
      {...rest}
      render={handleRender}
    />
  );
}
```

接下来则是我们的公共页，私有页和登录页：

```js
function PublicPage() {
  return <h3>Public</h3>;
}

function ProtectedPage() {
  return <h3>Protected</h3>;
}

function LoginPage() {
  let history = useHistory();
  let location = useLocation();

  // 判断当前 /login 的 state 中是否有 from，如果没有就设置为 "/"
  let { from } = location.state || { from: { pathname: "/" } };
  let login = () => {
    fakeAuth.authenticate(() => {
      // 通过History进行跳转到from的页面
      history.replace(from);
    });
  };

  return (
    <div>
      <p>You must log in to view the page at {from.pathname}</p>
      <button onClick={login}>Log in</button>
    </div>
  );
}
```

运行项目，你会发现点击 Protected Page 会自动跳转到登录页面，点击登录按钮后，又会自动跳转回 Protected Page.

## 页面不存在

如果用户访问不存在的路径，我们可以通过 Route 的匹配规则来捕捉所有不存在的路径：

```js
<Route path="*">
  <NoMatch />
</Route>
```







