配置菜单路由权限有几种方式：
1. 防君子不防小人
菜单栏虽然没有页面权限，但可以输入url访问没有权限的页面
2. 统统防着
这就需要动态路由

#### 实现动态菜单
待更新

#### 实现动态路由
实现思路是使用Umi的[运行时配置](https://umijs.org/zh-CN/docs/runtime-config)
提供了几个方法，其中`patchRender`和`render`配置配合使用，请求服务端根据响应动态更新路由

`patchRender({ routes })`：用于修改路由
`render(oldRender)`：用于渲染之前做权限校验

###### 实现步骤
1. `src`目录下新建`app.ts`
2. `app.ts`中重写`patchRender`，`render`方法
```js
// authRoutes用于暂存render请求的路由配置信息
let authRoutes: any[];

export function patchRoutes({ routes }) {
  authRoutes.forEach(route => routes.push(route));
}

export function render(oldRender) {
  fetch('/XXX/XXX')
    .then(res => res.json())
    .then(res => {
      authRoutes = res.result.menus;
      oldRender();
  });
}
```
但到此结束发现，页面时空白页，需要`parseRoutes`方法解析下数据，仍然写在`app.ts`中。
```js
function parseRoutes(authRoutes: any[]) {
  if (authRoutes) {
    return authRoutes.map(item => ({
      path: item.path,
      name: item.name,
    // umi的路径类似`@pages/Welcome`样式
    // 下面的模板字符串先加`@/`再去掉前两个字符看似多此一举，但require不支持变量引入，必须如此
    // 具体参考：https://blog.csdn.net/qq_36706941/article/details/111240146
      component: item.component ? require(`@/${item.component.slice(2)}`).default : undefined,
      routes: item.routes ? parseRoutes(item.routes) : undefined
    }));
  }
  return [];
}
```
改写`patchRoutes`。
```js
export function patchRoutes({ routes }) {
  parseRoutes(authRoutes).forEach(route => routes.push(route));
}
```