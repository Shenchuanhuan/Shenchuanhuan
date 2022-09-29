---
title: Can't resolve 'xxx' in 'xx'
date: 2022-09-29
tags: [Webpack]
---

如果：
1. 使用`webpack`配置了文件解析`loader`
2. 文件引入时（不写文件后缀）: `import A from '../src/test'`

遇到了这样的提示：`Can't resolve 'xxx' in 'xx'`，可以检查下`resolve.extensions`配置。

------------ 
### resolve.extensions
[webpack.resolve](https://webpack.js.org/configuration/resolve/#resolveextensions)

```javascript
    module.exports = {
        //...
        resolve: {
            extensions: ['.jsx', '.json']
        }
    }
```

配置了`extensions`后，引入文件时可以省略后缀名。
```javascript
    import File from '../path/to/file';
```
需要注意的是，如果被引入文件所在文件夹还存在同名不同后缀的文件，`webpack`会根据`extension`配置数组顺序(数组中较为先前的)来解析引入文件，其它则会被略过。
比如：
```javascript
    // extensions配置
    module.exports = {
        //...
        resolve: {
            extensions: ['.jsx', '.css', '.json', '...'];
        }
    };

    // 目录结构
    // - src
    //  - index.jsx
    //  - index.css
    //  - test.css
    //  - test.json

    // index.jsx
    import './index'; // webpack会认为是index.jsx，不能引入CSS
    import './index.css'; // 带后缀的全称，能够引入CSS
    import './test'; // 不带后缀，但.css配置在.json配置之前，因此会引入test.css而不是test.json
```
[上面🌰的地址](https://github.com/Shenchuanhuan/blog-demo/tree/main/webpack/extensions-demo)

‼️ 上面提到的`extensions`配置方式会覆盖掉默认的`extensions`配置。但是可以使用`'...'`来获取默认配置。
```javascript
    module.exports = {
        //...
        resolve: {
            extensions: ['.ts', '...'],
        },
    };
```