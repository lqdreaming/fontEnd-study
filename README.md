# 前端页面适配

### 一.两种布局适配方式

#### 1.响应式设计

在不同的分辨率下，都要让内容以最佳的展现形式呈现给你用户，提升用户体验。

##### **viewport设置**

具体可以查看[移动前端开发之viewport的深入理解](https://www.cnblogs.com/2050/p/3877280.html)

```
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

##### **Media Queries媒体查询**

断点系统-在chrome的Responsive Devices里面查看，各种设备的屏幕分辨率大小有几十种，但真正不会采用这么细致的断点设置。

![屏幕快照 2021-01-04 下午2.02.51](/Users/liuqi/Desktop/屏幕快照 2021-01-04 下午2.02.51.png)

```css
/*当页面宽度大于1000px且小于1200px的时候执行，1000-1200*/
@media screen and (min-width:1000px) and (max-width: 1200px){
    body{
        font-size:16px
    }
}
/*当页面宽度大于1280px且小于1366px的时候执行,1280-1366*/
@media screen and (min-width:1280px) and (max-width: 1366px){
    body{
        font-size:18px
    }
}
/*当页面宽度大于1440px且小于1600px的时候执行,1440-1600*/
@media screen and (min-width:1440px) and (max-width:1600px){
    body{
        font-size:20px
    }
}
/*当页面宽度大于1680px且小于1920px的时候执行,1680-1920*/
@media screen and (min-width:1680px) and (max-width:1920px){
    body{
        font-size:22px
    }
}
```

#### 2.自适应设计

<img src="https://www.w3cways.com/wp-content/uploads/2015/01/01_Responsive-vs-Adaptive.gif" alt="01_Responsive-vs-Adaptive" style="zoom:50%;" />

[1]百分比-以父元素为基准的，利用calc进行调整。

[2]flex弹性盒子(阮一峰的[Flex 布局教程：实例篇](http://www.ruanyifeng.com/blog/2015/07/flex-examples.html))和grid栅格布局([CSS Grid 网格布局教程](http://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html))

flex布局一般用的比较多，而grid使用比较少，主要在于二者的兼容性，[查看兼容性](https://caniuse.com/)：

flex兼容性：

![img](https://pic2.zhimg.com/80/v2-d694db089bdf0789636ca26804ac9bd5_1440w.jpg?source=1940ef5c)

grid兼容性

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190623123242923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzYwNjE1OA==,size_16,color_FFFFFF,t_70)

[3]rem和vw单位（小程序rpx）

**rem**：html标签的字体大小，即`1rem=html.style.fontSize`,默认为16px。

**vh and vw**：相对于视口的高度和宽度，而不是父元素的（CSS百分比是相对于包含它的最近的父元素的高度和宽度）。1vh 等于1/100的视口高度，1vw 等于1/100的视口宽度。

### 二.项目的适配方案

#### 1.Activity studio的适配方案

[1]media响应式-调整页面布局(例如因此侧边栏菜单)，字体大小，图片大小等。

```css
@media screen and (min-width: 1920px) {
  .no-zoom {
  }
}

@media screen and (min-width: 1440px) and (max-width: 1919px) {
  .no-zoom {
  }
}

@media screen and (min-width: 1200px) and (max-width: 1439px) {
  .no-zoom {
  }
  .zoom {
  }
}

@media screen and (max-width: 1199px) {
  .no-zoom {
  }
}
```

由于AS项目集成到edmodo，Microsoft Teams等平台上面，在这些平台内部适配，分辨率不一，所以需要对1440-1920px进行缩放适配。

[2]zoom缩放-PC端body宽度在1440-1920px之间，进行zoom动态缩放

```js
if (!mobileAndTabletCheck() && (isSafari || isChrome)) {
          var wW = window.innerWidth
          var designPCWidths = [1920, 1440]
          var designWidth = 1920
          for (var i = 0; i< designPCWidths.length; i++) {
            if (wW < designPCWidths[i] && designPCWidths[i] < designWidth) {
              designWidth = designPCWidths[i]
            }
          }
          var scale = (wW / designWidth)
          var bodyStyle = document.body.style
          bodyStyle.zoom = +(scale.toFixed(2))

          document.body.className += ' zoom'
        } else {
          document.body.className += ' no-zoom'
        }
 }
```

[3]scale缩放-PC端body里面的模块app进行scale动态缩放，保持宽度沾满，高度缩放

```js
   function adjustScale() {
      var app = document.querySelector('#app')
      var wH = window.innerHeight
      var wW = window.innerWidth
      var designPCWidths = [1920, 1440]
      var designMobileWidth = 992
      var designWidth = designMobileWidth
      if (!mobileAndTabletCheck()) {
        designWidth = 1920
        for (var i = 0; i< designPCWidths.length; i++) {
          if (wW < designPCWidths[i] && designPCWidths[i] < designWidth) {
            designWidth = designPCWidths[i]
          }
        }
      }
      var rate = (wW / wH).toFixed(2)
      var realWidth = designWidth
      var realHeight = parseInt(realWidth / rate)
      var scale = (wW / designWidth).toFixed(2)
      // fix white margin in mobile phone's iframe.
      scale = (+scale) + adjustScale
      app.style.width = realWidth + 'px'
      app.style.height = realHeight + 'px'
      setTimeout(function () {
        app.style.transform = 'scale(' + scale + ')'
        app.style.transformOrigin = 'left top'
      }, 100)
  }
```

#### 2.Edmodo适配方案

[1].整体采用reactstrap里面的布局-grid网格/flex适配

```html
<div class="row">
  <div class="col-lg-6">
  </div><!-- /.col-lg-6 -->
</div><!-- /.row -->
```

[2].media调整页面细节

```scss
$mobile-screen: 767.98px;
$tablet-screen: 991.98px;
$navbar_mobile_screen: 577px;

@mixin mobile{
  @media only screen and (max-device-width: $mobile-screen){
    @content;
  }
}
@mixin discover_mobile_landscape {
  @media only screen and (max-width: $mobile-screen) and (orientation: landscape) {
    @content;
  }
}
@mixin mobile_landscape {
  @media only screen and (max-device-width: $tablet-screen) and (orientation: landscape) {
    @content;
  }
}
@mixin tablet{
  @media only screen and (min-device-width: $mobile-screen) and (max-device-width: $tablet-screen){
    @content;
  }
}
@mixin mobile_and_tablet{
  @media only screen and (max-device-width: $tablet-screen){
    @content;
  }
}
@mixin navbar_tablet {
  @media only screen and (min-device-width: $navbar_mobile_screen) and (max-device-width: $tablet-screen){
    @content;
  }
}
```

综上：media方案能较好的还原在各屏幕尺寸的设计稿，但需要对每个页面进行UI的多端设计，相对于开发人员需要多套css样式，特别是对于适配的屏幕尺寸比较多的时候，同时项目还需要考虑用户对页面的伸缩时候，这样代码将非常冗杂，重复度和复杂度较高。

### 三手淘的适配方式和demo演示

#### 1.rem适配方案

[1] 在index.js引入[lib-flexible](https://github.com/amfe/lib-flexible):根据屏幕的宽度变化，对root的fontSize进行动态设置。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import 'lib-flexible'

ReactDOM.render(
​    <App />,
  document.getElementById('root')
);

serviceWorker.unregister();
```

[2] 安装postcss-px2rem-利用px2rem插件把css样式中的px单位自动转换为rem

config-overrides.js

```js
const { override, fixBabelImports, adjustStyleLoaders, addWebpackAlias, addLessLoader }  = require("customize-cra")
const path = require("path")
const rewirePostcss = require('react-app-rewire-postcss');
const px2rem = require('postcss-px2rem')

module.exports = override(
​    (config, env) => {
​        // 重写postcss
​        rewirePostcss(config,{
​            plugins: () => [
​                require('postcss-flexbugs-fixes'),
​                require('postcss-preset-env')({
​                    autoprefixer: { flexbox: 'no-2009' },
​                    stage: 3,
​                }),
​               //关键:设置px2rem
​                px2rem({
​                    remUnit: 75, // 以设计稿750px为标准设置
​                })
​            ],
​        });
​        return config
​    }
);
```

[3]对于不同dpr(dpr > 1)的retina屏幕的处理,由于安卓的dpr很复杂，没有考虑安卓的dpr，只考虑了iphone的兼容处理。

去掉index.html中的这行 <meta name="viewport" content="width=device-width, initial-scale=1" />

data-dpr会随不同的retina屏幕去动态设置。

或者保留<meta name="viewport" content="width=device-width, initial-scale=1" />，但需要动态设置meta进行缩放：

```js
const metaEl = document.querySelector('meta[name="viewport"]');
const dpr = window.devicePixelRatio || 1;
const scale = 1 / dpr;

// 设置viewport，进行缩放，达到高清效果
metaEl.setAttribute('content', 'width=' + dpr * docEl.clientWidth + ',initial-scale=' + scale + ',maximum-scale=' + scale + ', minimum-scale=' + scale + ',user-scalable=no');

// 设置data-dpr属性，留作的css hack之用
docEl.setAttribute('data-dpr', dpr);
```

这样就可以用rem来适配多端的页面，只需要一套样式代码，但有几个方面缺陷：

[4]行内样式没有转换-在babel-loader前自写一个jsxPx2RemLoader来转换行内代码中的px单位

reg.js-需要转换的行内样式的正则匹配

```js
// 匹配jsx中的px 如 1px
const pxReg = /\b(\d+(\.\d+)?)px\b/g;    

// 匹配jsx中 缩写形式的style 如：marginRight: 1
const styleReg = {
  marginTop: /\bmarginTop(?:\s+):(?:\s+)?(\d+)(px)?/g,
  marginRight: /\bmarginRight(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  marginBottom: /\bmarginBottom(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  marginLeft: /\bmarginLeft(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  fontSize: /\bfontSize(?:\s+)?:(?:\s+)?(\d+)(px)?(px)?/g,
  paddingTop: /\bpaddingTop(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  paddingRight: /\bpaddingRight(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  paddingBottom: /\bpaddingBottom(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  paddingLeft: /\bpaddingLeft(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  width: /\bwidth(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
  height: /\bheight(?:\s+)?:(?:\s+)?(\d+)(px)?/g,
}
 
// 匹配img 中的行内样式 width: '20'
const imgReg = {    
  height: /\bheight(?:\s+)?=(?:\s+)?(\'||\")?(\d+)?=(\'||\")?(px)?/g,
  width: /\bwidth(?:\s+)?=(?:\s+)?(\'||\")?(\d+)?=(\'||\")?(px)?/g,
}
// 匹配数字
const numReg = /(\d+)/g;
 
module.exports={
  pxReg,
  styleReg,
  imgReg,
  numReg,
}
```

jsxPx2RemLoader.js

```js
const _ = require('lodash');
const {
  pxReg,
  styleReg,
  imgReg,
  numReg,
} = require('./reg');
const loaderUtils = require('loader-utils');
// 默认参数
const defaultopts = {
    remUnit: 75, // rem unit value (default: 100)
    remFixed: 2, // rem value precision (default: 2)
}

module.exports = function(source) {
  // console.log(source, 99999)
  if (this.cacheable) {
    this.cacheable();
  }
  // 获取webpack配置好的参数
  const opts = loaderUtils.getOptions(this);
  // 将参数组合
  const config = Object.assign({}, defaultopts, opts);
  let backUp = source;
  
  if (pxReg.test(backUp)) {
    backUp = backUp.replace(pxReg, px => {
      let val = px.replace(numReg, num => {
        // 精确到几位
        const value = num / config.remUnit;
        return parseFloat(value.toFixed(config.remFixed));
      });
      val = val.replace(/px$/, 'rem');
      return val;
    });
  }
 
  // marginRight: 1 => marginRight: '0.01rem'
  _.each(styleReg, (reg, styleName) => {
    if (reg.test(backUp)) {
      backUp = backUp.replace(reg, val => {
        return val.replace(numReg, num => {
          const styleValue = parseFloat((num / config.remUnit).toFixed(config.remFixed));
          return `"${styleValue}rem"`;
        });
      });
    }
  });
 
  // img标签 width: 1 => style={{width: '0.01rem'}}
  _.each(imgReg, (reg, styleName) => {
    if (reg.test(backUp)) {
      backUp = backUp.replace(reg, val => {
        let style = '';
        val.replace(numReg, num => {
          const imgValue = parseFloat((num / config.remUnit).toFixed(config.remFixed));
          style = `${imgValue}rem`;
        });
        return `style={${styleName}:${style}}`;
      });
    }
  });
  
  return backUp;
}
```

config-overrides.js中增加loader的重写

```js
const { override, fixBabelImports, adjustStyleLoaders, addWebpackAlias, addLessLoader }  = require("customize-cra")
const path = require("path")
const rewirePostcss = require('react-app-rewire-postcss');
const px2rem = require('postcss-px2rem')

module.exports = override(
    (config, env) => {
        // 重写postcss
        rewirePostcss(config,{
            plugins: () => [
                require('postcss-flexbugs-fixes'),
                require('postcss-preset-env')({
                    autoprefixer: {
                        flexbox: 'no-2009',
                    },
                    stage: 3,
                }),
               //关键:设置px2rem
                px2rem({
                    remUnit: 75,
                })
            ],
        });
        // 修改 rule 配置 ，增加loader规则 (具体看配置)
        const loaders = config.module.rules.find(rule => Array.isArray(rule.oneOf));
        loaders.oneOf.splice(1, 1, {
            test: /\.(js|mjs|jsx|ts|tsx)$/,
            include: path.appSrc,
            exclude: /(node_modules)/,
            use: [
              {
                  loader: require.resolve('babel-loader'),
                  options: {
                      cacheDirectory: true,
                  }
              }, 
              {
                  loader: path.join(__dirname, 'loaders/jsxPx2RemLoader'),
                  options: {
                    remUnit: 75,
                    remFixed: 3,
                  }
              }
          ]
         })

        return config
    }
);
```

[5]行内动态计算的px无法转换-自写js函数进行转换

```js
// 给js调用的，某一dpr下rem和px之间的转换函数
const docEl = document.documentElement;

const dpr = window.devicePixelRatio || 1;
const rem = docEl.clientWidth * dpr / 10;
window.rem2px = function(v) {
	v = parseFloat(v);
	return v * rem;
};

window.px2rem = function(v) {
  v = parseFloat(v);
  return v / rem;
};
```

[6]与第三方UI框架冲突

安装postcss-px2rem-exclude忽略要转化的文件

```js
px2rem({
	remUnit: 75,
	exclude: /node_modules/i
})
```

如下图是对css样式，行内和动态样式的进行rem适配的demo演示：

<img src="/Users/liuqi/Desktop/adapt.gif" alt="adapt" style="zoom:125%;" />

#### 2.vw/vh适配

比rem适配容易，不需要像rem一样对根元素进行动态计算，但兼容性比rem稍差。

修改config-overrides.js

```js
const { override, fixBabelImports, adjustStyleLoaders, addWebpackAlias, addLessLoader, addPostcssPlugins, addWebpackPlugin }  = require("customize-cra")
const path = require("path")
const rewirePostcss = require('react-app-rewire-postcss');
const px2rem = require('postcss-px2rem-exclude')
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;
  
// 引入postCss插件
const postcssNormalize = require('postcss-normalize');
const postcssAspectRatioMini = require('postcss-aspect-ratio-mini');
const postcssPxToViewport = require('postcss-px-to-viewport');
const postcssWriteSvg = require('postcss-write-svg');
const postcssCssnext = require('postcss-cssnext');
const postcssViewportUnits = require('postcss-viewport-units');
const cssnano = require('cssnano');

module.exports = override(
    // 针对antd 实现按需打包：根据import来打包 (使用babel-plugin-import)  
    fixBabelImports("import", {    
        libraryName: "antd",    
        libraryDirectory: "es",    
        style: true, // 自动打包相关的样式 默认为 style:'css'  
    }),
     // 配置指定文件为sass全局文件，可以不用导入就可以使用
     adjustStyleLoaders(rule => {
        if (rule.test.toString().includes('scss')) {
            rule.use.push({
                loader: require.resolve('sass-resources-loader'),
                options: {
                    resources: [
                        './src/assets/_vars.scss',
                        './src/assets/_mixin.scss',
                        './src/assets/_func.scss',
                        './src/assets/_public.scss'
                    ]
                }
            });
        }
    }),
    addPostcssPlugins([    
        require("autoprefixer")(),    
        postcssAspectRatioMini(),   
        postcssPxToViewport({      
            viewportWidth: 750, // (Number) The width of the viewport.     
            viewportHeight: 1334, // (Number) The height of the viewport.    
            unitPrecision: 5, // (Number) The decimal numbers to allow the REM units to grow to.  
            viewportUnit: "vw", // (String) Expected units.     
            selectorBlackList: [], // (Array) The selectors to ignore and leave as px.  
            minPixelValue: 1, // (Number) Set the minimum pixel value to replace.     
            mediaQuery: true, // (Boolean) Allow px to be converted in media queries.   
            exclude: [/(node_modules)/],
        }), 
        postcssWriteSvg({      
            utf8: false   
        })
    ]),  process.argv[2]==='analysis' && addWebpackPlugin(new BundleAnalyzerPlugin()), // 在build之前可以先对包的大小进行分析，如果太大在进行优化);

    (config, env) => {
        const loaders = config.module.rules.find(rule => Array.isArray(rule.oneOf));
        loaders.oneOf.splice(1, 1, {
            test: /\.(js|mjs|jsx|ts|tsx)$/,
            include: path.appSrc,
            exclude: /(node_modules)/i,
            use: [
              {
                  loader: require.resolve('babel-loader'),
                  options: {
                      cacheDirectory: true,
                  }
              }, 
              {
                  loader: path.join(__dirname, 'loaders/jsxPx2VwLoader'),
                  options: {
                    viewportWidth: 750,
                    unitPrecision: 5
                  }
              }
          ]
         })
        return config
    }
);
```

jsxPx2VwLoader.js-把rem的计算函数换为vw的计算函数即可：用于解决行内样式转换为vw的问题

```js
const loaderUtils = require('loader-utils');
const _ = require('lodash');
const {
    pxReg,
    styleReg,
    imgReg,
    numReg,
} = require('./reg');

function createPxReplace (num, viewportSize, minPixelValue, unitPrecision) {
     var pixels = parseFloat(num)
     if (pixels <= minPixelValue) return
     return toFixed((pixels * 100 / viewportSize ), unitPrecision)
}

function toFixed (number, precision) {
    var multiplier = Math.pow(10, precision + 1),
     wholeNumber = Math.floor(number * multiplier)
    return Math.round(wholeNumber / 10) * 10 / multiplier
}

const defaultopts = {
    unitToConvert: 'px',
    viewportWidth: 750,
    unitPrecision: 5,
    viewportUnit: 'vw',
    fontViewportUnit: 'vw',
    minPixelValue: 1
};

module.exports = function(source) {
    // console.log(source, 99999)
    if (this.cacheable) {
      this.cacheable();
    }
    // 获取webpack配置好的参数
    const opts = loaderUtils.getOptions(this);
    // 将参数组合
    const defaults = Object.assign({}, defaultopts, opts);
    let backUp = source;
    
    if (pxReg.test(backUp)) {
        backUp = backUp.replace(pxReg, px => {
          let val = px.replace(numReg, num => {
            // 精确到几位
            const value = createPxReplace(num, defaults.viewportWidth, defaults.minPixelValue, defaults.unitPrecision);
            console.log(num, value, 'value9999----')
            return value;
          });
          val = val.replace(/px$/, defaults.viewportUnit);
          return val;
        });
      }
   
   
    // marginRight: 1 => marginRight: '0.01rem'
    _.each(styleReg, (reg, styleName) => {
      if (reg.test(backUp)) {
        backUp = backUp.replace(reg, val => {
          return val.replace(numReg, num => {
            const styleValue = createPxReplace(num, defaults.viewportWidth, defaults.minPixelValue, defaults.unitPrecision, defaults.viewportUnit);
            return `"${styleValue}${defaults.viewportUnit}"`;
          });
        });
      }
    });
   
   
    // img标签 width: 1 => style={{width: '0.01rem'}}
    _.each(imgReg, (reg, styleName) => {
      if (reg.test(backUp)) {
        backUp = backUp.replace(reg, val => {
          let style = '';
          val.replace(numReg, num => {
            const imgValue = createPxReplace(num, defaults.viewportWidth, defaults.minPixelValue, defaults.unitPrecision, defaults.viewportUnit);
            style = `${imgValue}${defaults.viewportUnit}`;
          });
          return `style={${styleName}:${style}}`;
        });
      }
    });
    
    return backUp;
  }
```

行内动态计算的px无法转换-同样自写js函数进行转换-和jsxPx2VwLoader.js中的createPxReplace函数类似

<img src="/Users/liuqi/Desktop/vw.gif" alt="vw" style="zoom:150%;" />

3.rem和vw兼容性

<img src="https://img-blog.csdnimg.cn/20191016095813174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjc3NDA0,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:150%;" />![在这里插入图片描述](https://img-blog.csdnimg.cn/20191016095835586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjc3NDA0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191016095835586.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMjc3NDA0,size_16,color_FFFFFF,t_70)

VW的完全兼容需要Android4.4+和ios8+,而rem的完全兼容需要Android2.1+和ios6+。
