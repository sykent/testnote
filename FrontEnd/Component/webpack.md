# webpack3

## 什么是webpack？
![webpack.png](https://i.loli.net/2017/08/23/599d7a5b079d1.png)

webpack = module building system

在webpack看来，所有的资源文件都是模块(module)，只是处理的方式不同。

关于webpack的使用，其实和我们平时开发业务产品是一个道理。

产品需求 => 代码设计 => 提供API给开发者使用。

webpack解决的需求点就是如何更好的加载前端模块，webpack本身并不关心这个模块是什么，它只是调度配置文件中对模块处理的方式来完成这一切,而不必纠结文件类型。

比如我们会在项目中使用ES6/7/8的语法来编写JS代码，于是我们只需要配置babel-loader来转换到ES5从而实现兼容。

比如我们需要加载HTML文件获取HTML字符串，于是我们只需要配置raw-loader即可拿到对应文件的字符串。

比如我们需要将SASS/LESS文件预编译成CSS，于是我们只需要配置sass-loader/less-loader即可处理。

## webpack与gulp的区别？
**gulp 是 task runner，Webpack 是 module bundler。**

gulp 最核心的功能：

1. 任务定义和组织
2. 基于文件 stream 的构建
3. 插件体系

webpack 最核心的功能：

1. 按照模块的依赖构建目标文件
2. loader 体系支持不同的模块
3. 插件体系提供更多额外的功能

## webpack3优化实践
在用Vue-cli生成的项目中已经自带webpack配置文件，我们只需要运行`npm run build`就能打包文件，但是默认配置打包出来的文件通常过大，这就需要我们改动配置来优化。

### 图解打包文件
在`package.json`中加入如下脚本
```json
"scripts": {
    "analyze": "npm_config_preview=true  npm_config_report=true node build/build.js"
},
```
运行`npm run analyze`得到分析大小和分析图

![webpacktotal0.png](https://i.loli.net/2017/08/23/599d799d39782.png)

分析图中JS文件的大小

![webpacksize0.png](https://i.loli.net/2017/08/23/599d799d30e2a.png)

分析图

![webpackanalyze.png](https://i.loli.net/2017/08/23/599d799d458ba.png)

### 删去或替换代价大的模块

我们看到echart模块占了将近四分之一的体积，但是好像项目中并没有用到，于是定位并删除模块引用。

![echart.png](https://i.loli.net/2017/08/23/599d7a846831a.png)

还发现fontawesome图标库模块在打包过程中被webpack标记为big，先以`fa fa`关键词定位

![icon0.png](https://i.loli.net/2017/08/23/599d7a846765a.png)

在项目中查看这个图标的具体应用

![menucollapse0.png](https://i.loli.net/2017/08/23/599d799cd4852.png)

花费400KB+大小的图标库来使用一个图标有点得不偿失，寻找element自带图标替代方案

![menucollapse1.png](https://i.loli.net/2017/08/23/599d799ce5aaf.png)
![menucollapse3.png](https://i.loli.net/2017/08/25/59a0216fdae0f.png)

替代完成后，删除import引用

![importfontawesome.png](https://i.loli.net/2017/08/23/599d7a8465aeb.png)

### Code Splitting
对于大型的web应用而言，把所有的代码放到一个文件的做法效率很差，特别是在加载了一些只有在特定环境下才会使用到的阻塞的代码的时候。Webpack有个功能会把你的代码分离成Chunk，后者可以按需加载。

Code Spliting的具体做法就是一个分离点，在分离点中依赖的模块会被打包到一起，可以异步加载。一个分离点会产生一个打包文件。 

Code Spliting用到了webpack自带的`CommonsChunkPlugin`插件，在webpack的配置文件中这样添加。

![webpack-codesplit.png](https://i.loli.net/2017/08/31/59a7d496afb54.png)

按需加载在代码中的运用
```javascript
exportExcel() {
    require.ensure([], () => {
        const { export_json_to_excel } = require('../../../vendor/exportToExcel');
        const tHeader = ['序号', '进件渠道', '产品名称', '客户名称', '放款日期', '合同金额', '期次', '期限', '起息日', '还款日', '应还本金', '应还费用', '应还利息', '应还罚息', '待抵扣保证金', '应还合计', '是否逾期', '实际还款日期'];
        const filterVal = ['index', 'finaCoopCompNo', 'name', 'customerName', 'loanDate', 'demandAmt', 'cycleNum', 'term', 'loanDate', 'dueDate', 'principal', 'fee', 'interest', 'penalizedAmt', 'despoitAmt', 'sumFee', 'isOverdue', 'factDueDate'];
        let repayDetailList = this.repayDetailList;
        const data = this.formatJson(filterVal, repayDetailList);
        export_json_to_excel(tHeader, data, '还款明细');
    })
},
```
这样xlsx（导出excel）模块就开启了按需加载，进一步降低了页面请求中需要JS的体积。
### GZIP压缩
gzip对于JS和CSS等文件压缩比很大，能进一步压缩用户访问服务器需要的体积

先安装压缩插件`npm install --save-dev compression-webpack-plugin`

在配置中设置`productionGzip: true`

![webpackgzip.png](https://i.loli.net/2017/08/23/599d873a15fc3.png)

### webpack3兼容性问题（JS压缩）

webpack2升级到webpack3的兼容性问题，JS压缩插件需要升级并重新在配置中应用

`npm install uglifyjs-webpack-plugin@beta --save-dev`

![webpackjscompress.png](https://i.loli.net/2017/08/23/599d8989c789a.png)

### 重新打包
`npm run analyze`或`npm run build`

![webpacktotal1.png](https://i.loli.net/2017/08/23/599d799d37624.png)

![webpack1.png](https://i.loli.net/2017/08/23/599d799d3abb3.png)

![webpackgzip1.png](https://i.loli.net/2017/08/23/599d799d26128.png)

**打包大小由3MB+ => 1.7MB => 500KB（Nginx开启gzip）**