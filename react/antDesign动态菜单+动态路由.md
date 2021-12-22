配置菜单路由权限有几种方式：
1. 防君子不防小人
菜单栏虽然没有页面权限，但可以输入url访问没有权限的页面
2. 统统防着
这就需要动态路由

#### 一、实现动态菜单
待更新

#### 二、实现动态路由
实现思路是使用Umi的[运行时配置](https://umijs.org/zh-CN/docs/runtime-config)
提供了几个方法，其中`patchRender`和`render`配置配合使用，请求服务端根据响应动态更新路由

`patchRender({ routes })`：用于修改路由
`render(oldRender)`：用于渲染之前做权限校验

##### 基本步骤
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
但到此结束发现，页面是空白页，需要`parseRoutes`方法解析下数据，仍然写在`app.ts`中。
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

##### 登录后动态获取路由
基本思路是在登录后获取路由，再强制刷新一下，由patchRoutes修改路由。
获取路由后的处理,设置`sessionStorage`的`LOADED`为标志，若非已加载就暂存`MENUS`
```js
// 请求路由接口：返回res
if (sessionStorage.getItem('LOADED') !== '1') {
  sessionStorage.setItem('LOADED', '1');
  sessionStorage.setItem('MENUS', JSON.stringify(res.result.menus));
  window.location.reload();
}
```

`app.ts`中`render`方法需要修改
```js
export function render(oldRender: () => void) {
  const menus = JSON.parse(sessionStorage.getItem('MENUS') ?? '[]');
  if (menus?.length > 0) {
    authRoutes = menus;
    sessionStorage.removeItem('MENUS');
    oldRender();
  } else {
    oldRender();
  }
}
```

这样是实现了登录后动态修改路由，但还需要登出后删除`LOADED`
```js
sessionStorage.removeItem('LOADED');
```

##### 此外，还有一种方法
但是，试过后发现菜单被锁死，虽然URL变了但是没有跳转页面，遂放弃。
原文：[如何在 umijs 中手动触发运行时配置 patchRoutes？](https://segmentfault.com/q/1010000038188612?utm_source=tag-newest)
```js
let extraRoutes;
export function patchRoutes({ routes }) {
  routes[1].routes = extraRoutes.concat(routes[1].routes);
}
export function render(oldRender) {
   window.umi_reload = ()=>{
      //根据用户请求菜单数据
      await xxx //读取数据放入extraRoutes
      oldRender();
   }
   window.umi_reload();
}
```


#### 参考文章
[如何用ant design 实现动态路由？ - 锦城牛仔的回答 - 知乎](https://www.zhihu.com/question/336131537/answer/2174495262)
[Antd pro 动态路由 - 哔哩哔哩](https://www.baidu.com/link?url=A690myL7vy0srpOQ76eP_7vjESstK2XkJD2tI6YNM8fZCPu230MtfMkNZ0oTOPMlovlNvMpBeRqZXuSKPjxG5a&wd=&eqid=a151dcad0002c0f50000000461c01938)