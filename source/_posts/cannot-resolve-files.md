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
webpack会按配置顺序来解析相应后缀文件，如果有同名但后缀不同的文件，只会解析后缀是`extension`配置数组中第一个配置的文件，其它的同名文件则会跳过。

‼️ 上面提到的`extensions`配置方式会覆盖掉默认的`extensions`配置。但是可以使用`'...'`来获取默认配置。
```javascript
    module.exports = {
        //...
        resolve: {
            extensions: ['.ts', '...'],
        },
    };
```