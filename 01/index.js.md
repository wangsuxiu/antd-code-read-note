<!--
 * @Author: wsx
 * @Date: 2021-02-03 17:40:50
 * @LastEditTime: 2021-02-04 16:52:30
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: /antd-code-read-note/01/index.js.md
-->
# index.js
### 前置知识
**require.context** 实现"路由"去中心化管理
> 官方解释
```

假设我们有如下目录:
example_directory
│
└───template
│   │   table.ejs
│   │   table-row.ejs
│   │
│   └───directory
│       │   another.ejs
```
当调用 ``` require ()```时
```
require('./template' + name + '.ejs');
```
此时webpack会解析```require() ```调用，提取出如下一些信息
```
Dicorector: ./template
Regular expression: /^.*\.ejs$/
```
##### 生成context module
翻译成上下文模块。它包含目录下所有模块的引用，如果一个request符合正则，那么它将会被require进来。

### require.context
创建自己的context。
可以给这个函数传入三个参数：一个要搜索的目录，一个标记表示是否还搜索其子目录，以及一个匹配文件的正则表达式。
webpack会在构建中解析代码中的``` require.context（）```。
语法如下:
```
require.context(directory, useSubdirectories = true, regExp = /^\.\/.*$/, mode= 'sync');
```
#### 示例

```
require.context('./test', false, /\.test\.js$/);
//（创建出）一个 context，其中文件来自 test 目录，request 以 `.test.js` 结尾。
```
##### 使用require.context改造route.js文件
###### 改造前
```
// rootRoute.js
const rootRoute = {
    childRoutes: [
        {
            path: '/',
            component: AppLayout,
            childRoutes: [
                require('./modules/shop/route'), //购买详情页
                require('./modules/order/route'), // 订单页
                require('./modules/login/route'), // 登录注册页
                require('./modules/service/route'), // 服务中心
                // ...
                // 其他大量新增路由
                // ...
            ]
        }
    ]
};
```

###### 改造后
```
const rootRoute = {
    childRoutes: [
        {
            path: '/',
            component: AppLayout,
            childRoutes: (r => {
                return r.keys().map(key => r(key));
            })(require.context('./', true, /^\.\/modules\/((?!\/)[\s\S])+\/route\.js$/))
        }
    ]
};

```
> exports和module.exports的区别 
1. module.exports初始值为一个空对象{}
2. exports是指向module.exports的引用
3. require()返回的是module.exports而不是exports

### index.ts源码解读
```
  // index.js
  function camelCase(name) {
    return name.charAt(0).toUpperCase() +
      name.slice(1).replace(/-(\w)/g, (m, n) => {
        return n.toUpperCase();
      });
  }

  // 抛出样式 这个正则是匹配当前目录下的所有的/style/index.tsx文件
  const req = require.context('./components', true, /^\.\/[^_][\w-]+\/style\/index\.tsx?$/);

  req.keys().forEach((mod) => {
    let v = req(mod);
    if (v && v.default) {
      v = v.default;
    }
    // 抛出组件 这个正则是匹配当前目录下的素有index.tsx文件
    const match = mod.match(/^\.\/([^_][\w-]+)\/index\.tsx?$/);
    if (match && match[1]) {
      if (match[1] === 'message' || match[1] === 'notification') {
        // message & notification should not be capitalized
        exports[match[1]] = v;
      } else {
        exports[camelCase(match[1])] = v;
      }
    }
  });
  module.exports = require('./components');

```
