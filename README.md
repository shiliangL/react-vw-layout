### react移动端适配方案 

> 本文项目基于create-react-app构建。更多移动端资料推荐

- [w3cplus 再聊移动端页面的适配](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)

- [w3cplus 如何在Vue项目中使用vw实现移动端适配](https://www.w3cplus.com/css/vw-for-layout.html)

- [w3cplus 使用Flexible实现手淘H5页面的终端适配](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)

- [本项目配置源码地址](https://github.com/shiliangL/react-vw-layout)
> tips

```css
/* 处理 vm 适配图片不显示问题 */
img {
  content: normal !important; 
}
```

### 1️⃣ 项目初始 + 暴露配置项

- 1、create-react-app react-vw-layout 初始化项目
- 2、npm run eject 暴露配置项

### 2️⃣ postCss插件安装配置

> package.json 中添加依赖,并安装

```js

"postcss-aspect-ratio-mini": "0.0.2",
//--坑点 已经更新 postcss-preset-env 所以请使用 "postcss-preset-env": "6.0.6"👇
"postcss-cssnext": "^3.1.0",
"postcss-flexbugs-fixes": "3.2.0",
"postcss-loader": "2.0.8",
"postcss-px-to-viewport": "0.0.3",
"postcss-viewport-units": "^0.1.4",
"postcss-write-svg": "^3.0.1"

```

> config/webpack.config.dev.js 文件中修改添加配置（开发环境生效）
> (生产环境打包配置如下，也是一样的在文件夹 config/webpack.config.prod 中修改)

```js
// config/webpack.config.dev.js
// 文件头部引进依赖
const fs = require('fs');
const path = require('path');
const resolve = require('resolve');
const webpack = require('webpack');
const PnpWebpackPlugin = require('pnp-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CaseSensitivePathsPlugin = require('case-sensitive-paths-webpack-plugin');
const InterpolateHtmlPlugin = require('react-dev-utils/InterpolateHtmlPlugin');
const WatchMissingNodeModulesPlugin = require('react-dev-utils/WatchMissingNodeModulesPlugin');
const ModuleScopePlugin = require('react-dev-utils/ModuleScopePlugin');
const getCSSModuleLocalIdent = require('react-dev-utils/getCSSModuleLocalIdent');
const getClientEnvironment = require('./env');
const paths = require('./paths');
const ManifestPlugin = require('webpack-manifest-plugin');
const ModuleNotFoundPlugin = require('react-dev-utils/ModuleNotFoundPlugin');
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin-alt');
const typescriptFormatter = require('react-dev-utils/typescriptFormatter');

// 移动端适配添加 - 插入
const postcssAspectRatioMini = require('postcss-aspect-ratio-mini');
const postcssPxToViewport = require('postcss-px-to-viewport');
const postcssWriteSvg = require('postcss-write-svg');
const postcssCssnext = require('postcss-preset-env'); //这个插件已经更新 postcss-preset-env 所以请使用 "postcss-preset-env": "6.0.6",
const postcssViewportUnits = require('postcss-viewport-units');
const cssnano = require('cssnano');


//配置项中添加使用

 {
    // Options for PostCSS as we reference these options twice
    // Adds vendor prefixing based on your specified browser support in
    // package.json
    loader: require.resolve('postcss-loader'),
    options: {
      // Necessary for external CSS imports to work
      // https://github.com/facebook/create-react-app/issues/2677
      ident: 'postcss',
      plugins: () => [
        require('postcss-flexbugs-fixes'),
        require('postcss-preset-env')({
          autoprefixer: {
            flexbox: 'no-2009',
          },
          stage: 3,
        }),

        // -----插入适配移动端配置项-----👇
        postcssAspectRatioMini({}),
        postcssPxToViewport({
          viewportWidth: 750, // (Number) The width of the viewport.
          viewportHeight: 1334, // (Number) The height of the viewport.
          unitPrecision: 3, // (Number) The decimal numbers to allow the REM units to grow to.
          viewportUnit: 'vw', // (String) Expected units.
          selectorBlackList: ['.ignore', '.hairlines'], // (Array) The selectors to ignore and leave as px.
          minPixelValue: 1, // (Number) Set the minimum pixel value to replace.
          mediaQuery: false // (Boolean) Allow px to be converted in media queries.
        }),
        postcssWriteSvg({
          utf8: false
        }),
        postcssCssnext({}),
        postcssViewportUnits({}),
        cssnano({
          //旧的 --坑点
          // preset: "advanced",
          // autoprefixer: false,
          // "postcss-zindex": false
          //新配置继续使用高级配置,按照这个配置
          "cssnano-preset-advanced": {
            zindex: false,
            autoprefixer: false
          },
        })
      ],
    },
},

```

### 3️⃣测试验证

> 修改 App.css 文件

```css
.App {
  width: 750px;
  height: 300px;
  background: #409eff;
  color: #ffffff;
  line-height: 200px;
  text-align: center;
}
```

> npm start 启动项目，打开控制台，这个时候已经生效了

![](https://user-gold-cdn.xitu.io/2018/12/22/167d3ba8d648eb46?w=3008&h=1656&f=png&s=307506)

> 配置好生产环境之后验证,npm run build,这个时候，生产打包已经生效了。

![](https://user-gold-cdn.xitu.io/2018/12/22/167d3c1fc05d283d?w=2342&h=1074&f=png&s=373052)

### 4️⃣ 一些兼兼容性hacks处理

> 修改 public/index.html, 引入阿里的 cdn

```html
<!-- index.html body 后添加-->
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
<script>
  window.onload = function() {
    window.viewportUnitsBuggyfill.init({
      hacks: window.viewportUnitsBuggyfillHacks
    });

    // 验证输出
    const winDPI = window.devicePixelRatio;
    const uAgent = window.navigator.userAgent;
    const screenHeight = window.screen.height;
    const screenWidth = window.screen.width;
    const winWidth = window.innerWidth;
    const winHeight = window.innerHeight;

    console.log(winDPI, "设备 DPI");
    console.log(uAgent, "客户端");
    console.log(screenWidth, "屏幕宽度");
    console.log(winHeight, "屏幕高度");
    console.log(winWidth, "Windows Width");
    console.log(winHeight, "Windows Height");
  };
</script>
```

![](https://user-gold-cdn.xitu.io/2018/12/22/167d3e154ebd9eca?w=3236&h=1640&f=png&s=205474)

> 至此配置算是完成了,更多其中插件的配置,还需要了解一下官方的配置使用,此外插件和可行性方案都会更新,这里只是作为自己学习实际的一个可行性方案实践的过程,如果有更多更新方案可以留言告知。谢阅~
