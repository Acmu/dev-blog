# 引入 Lodash 的最佳方式

Lodash 是我们很常用的一个库（npm 周下载量是 2.8 亿次，react 是 0.8 亿次，webpack 是 1.1 亿次），但很多人引入  Lodash 并不优雅（打包后的体积过大），本文就带大家来看看  Lodash 常用的几种引入方式。

先写结论，想看细节的话，可以继续。

## 结论

1. 直接写  `import _ from 'lodash';`  或  `import { isEqual } from 'lodash';`  引入的是整个  Lodash 包，约 70kb 左右（压缩后），`import isEqual from 'lodash/isEqual';`  引入的单个使用到的文件，约 15kb 左右（压缩后）。
1. 对于已有的项目，很多代码都是前两种写法的，那也不能一个一个的去改吧，是的，可以使用  `babel-plugin-lodash`  来帮助我们把前两种 变成第三种写法。
1. 对于新写的代码，推荐使用  `lodash-es` 包，因为这可以省去  babel 编译的时间。

## 直接引入

```javascript
// one.js
import _ from 'lodash';

console.log(_.isEqual(NaN, NaN));
```

build 后生成的 js 大小如下： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1585807319376-15744c89-e01c-461e-872e-51ad5df81e6a.png#align=left&display=inline&height=102&name=image.png&originHeight=204&originWidth=790&size=39865&status=done&style=none&width=395)

## 解构引入

```javascript
// two.js
import { isEqual } from 'lodash';

console.log(isEqual(NaN, NaN));
```

build 后如下： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1585807417167-a5708610-4720-4187-aada-0dce04a6c093.png#align=left&display=inline&height=69&name=image.png&originHeight=138&originWidth=691&size=29525&status=done&style=none&width=345.5) 可以看到，都是 72 kb。

## 具体文件引入

```javascript
// three.js
import isEqual from 'lodash/isEqual';

console.log(isEqual(NaN, NaN));
```

build 后如下： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1585807530012-9f9904f0-c101-4f35-a791-0324f7e0382e.png#align=left&display=inline&height=70&name=image.png&originHeight=139&originWidth=698&size=28472&status=done&style=none&width=349)

咦，这个时候我们看到了神奇的事情，打包后只有 15 kb，这就说明 webpack 只打包了用到的代码，其他没用到的并没有打包进去。 [说明文档](https://github.com/lodash/lodash#installation)也是这样指出的： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1585807782417-c2b74f84-25bd-485e-9a8b-2f5dd0c066d7.png#align=left&display=inline&height=290&name=image.png&originHeight=580&originWidth=1510&size=116236&status=done&style=none&width=755)

那我们就一直这样引入吧，还能做到按需加载，岂不是很 happy，其实不然，当我们引入的方法多了之后，就会变成如下这样：

```javascript
import isEqual from 'lodash/isEqual';
import chunk from 'lodash/chunk';
import debounce from 'lodash/debounce';
import throttle from 'lodash/throttle';
// ...
```

这样就写了很多重复的代码，明显没有解构写方便：`import { isEqual, chunk, debounce, throttle } from 'lodash';`  当然  Lodash 这么牛逼的库不可能想不到这个问题，所以有  babel-plugin-lodash 可以使用，它做的就是一些代码转换，如下：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1585808819405-11cc8433-d028-4f29-af95-7be1edcfa35b.png#align=left&display=inline&height=428&name=image.png&originHeight=856&originWidth=762&size=78861&status=done&style=none&width=381) 这个  babel 插件可以让以上 2 种方式都转成 文件引入。棒棒，我们来试一下：

```javascript
// four.js
import _ from 'lodash';
import { add } from 'lodash/fp';
import isEqual from 'lodash/isEqual';

const addOne = add(1);
_.map([1, 2, 3], addOne);
console.log(isEqual(NaN, NaN));
```

当我们没有使用  babel-plugin-lodash 时，build 后如下： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1585809274667-212a35f7-f457-48d3-8f37-7530c0e5c652.png#align=left&display=inline&height=69&name=image.png&originHeight=138&originWidth=705&size=28489&status=done&style=none&width=352.5)

167kb，好大。。。用了之后呢？神奇的事情发生了，一样的代码打包之后只有 48kb，nb（antd 也有一个类似的插件，是 [babel-plugin-import](https://github.com/ant-design/babel-plugin-import)，大家可以试一下） ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1586182329456-7abb7e3b-f0fd-4678-8953-42a003cc4eb1.png#align=left&display=inline&height=169&name=image.png&originHeight=338&originWidth=1250&size=73606&status=done&style=none&width=625)

只需要在 webpack 中添加如下代码即可：

```javascript
module: {
  rules: [
    // other ...
    {
      loader: 'babel-loader',
      test: /\.js$/,
      exclude: /node_modules/,
      query: {
        plugins: ['lodash'],
        presets: [['@babel/env', { targets: { node: 6 } }]],
      },
    },
  ],
},
```

如果你的项目代码中，有很多不规范的 import 写法，用这个方法非常友好，但如果你新写的项目，还是推荐你使用如下方式：

## 使用 es 引入

查看 Lodash 源码的 tag，你会发现有一个 es 分支： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1586183280612-f2ca5030-16e8-439e-9981-53ed39f1dba6.png#align=left&display=inline&height=371&name=image.png&originHeight=742&originWidth=804&size=70413&status=done&style=none&width=402)

切换到这个 tag 查看代码，如下： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1586183409747-991a986d-7241-444d-b87e-4ea1ec5d0b33.png#align=left&display=inline&height=211&name=image.png&originHeight=422&originWidth=1498&size=75985&status=done&style=none&width=749)

可以看到都是使用 ES Module 的模式引入的，因为 webpack 是一个模块打包器，所以她可以直接解析  ES Module 的代码，所以使用如下引用方式，也可以减小体积：（[lodash-es](https://www.npmjs.com/package/lodash-es) 是一个单独的包；由于这里写都是 var 变量等兼容性很好的代码，所以不需要 babel 编译；推荐看 Lodash 源码的话，使用这个包）

```javascript
// five.js
import { isEqual, map } from 'lodash-es';

console.log(isEqual(NaN, NaN));

map([1, 2, 3], console.log);
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1586184112588-0a7fc628-6033-4abc-9f9a-742ec86778c1.png#align=left&display=inline&height=100&name=image.png&originHeight=200&originWidth=752&size=36458&status=done&style=none&width=376)

最终，只有 16kb，也很酷！

我是推荐使用 es 引入的，因为这可以减少 babel 编译的时间。（当然这只对于支持 ES Module 语法的才行，如果是 node 使用，还是乖乖地 require 具体文件吧，不过 node 也快要支持 ES Module 了）

欢迎光临  [我的 GitHub](https://github.com/Acmu)、[我的 掘金](https://juejin.im/user/5bcab884e51d450e81091745)、我的公众号： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/197018/1586189330099-afc78321-662b-4b28-86fb-9211eb0fdacb.png#align=left&display=inline&height=310&name=image.png&originHeight=500&originWidth=900&size=95117&status=done&style=none&width=557)
