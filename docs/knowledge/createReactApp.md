# 非脚手架方式构建 React 项目

> 学习[从头开始搭建一个 react + typescript 脚手架项目](https://juejin.cn/post/7037316769195196429)博客,
> [本项目源码地址](https://github.com/ducaoya/create-react-app)

## 初始化 npm 项目

运行`npm init`

生成 package.json 文件

```json
{
  "name": "creat-react-app",
  "version": "1.0.0",
  "description": "非脚手架方式构建 React 项目",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "ducaoya",
  "license": "ISC"
}
```

## 配置 babel

创建.babelrc.js 文件

```js
// .babelrc.js
module.exports = {
  presets: [
    [
      '@babel/preset-env', //据设置的目标浏览器环境找出所需的插件去转译 ES6+ 语法
      {
        modules: false,
      },
    ],
    '@babel/preset-react', //支持react
    '@babel/preset-typescript', //支持ts
  ],
  plugins: [
    [
      '@babel/plugin-transform-runtime', //解决一些新特性，比如 includes
      {
        corejs: {
          version: 3,
          proposals: true,
        },
      },
    ],
  ],
};
```

## 配置 webpack

新建 config 文件夹，并且安装`webpack-merge`依赖

- 在 config 文件夹下创建 constant.js 定义公用变量

```js
const path = require('path');

const PROJECT_PATH = path.resolve(__dirname, '../'); // 当前项目根目录
const PROJECT_NAME = path.parse(PROJECT_PATH).name;
const isDev = process.env.NODE_ENV !== 'production';
const SERVER_HOST = '127.0.0.1';
const SERVER_PORT = 3000;

module.exports = {
  isDev,
  PROJECT_PATH,
  PROJECT_NAME,
  SERVER_HOST,
  SERVER_PORT,
};
```

- 在 config 文件夹下创建 webpack.common.js 文件存储公共配置

```js
const { resolve } = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const ForkTsCheckerWebpackPlugin = require('fork-ts-checker-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const WebpackBar = require('webpackbar');
const { isDev, PROJECT_PATH } = require('./constant');

module.exports = {
  entry: {
    index: resolve(PROJECT_PATH, './src/index.tsx'),
  },
  output: {
    filename: `js/[name]${isDev ? ' ' : '.[hash:8]'}.js`,
    path: resolve(PROJECT_PATH, './dist'),
  },
  optimization: {},
  resolve: {
    extensions: ['.tsx', '.ts', '.js', '.json'], // webpack 会按照定义的后缀名的顺序依次处理文件
    alias: {
      Src: resolve(PROJECT_PATH, './src'),
      Components: resolve(PROJECT_PATH, './src/components'),
      Utils: resolve(PROJECT_PATH, './src/utils'),
    },
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          isDev ? 'style-loader' : MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: false, // 默认就是 false
              sourceMap: isDev, // 开启后与 devtool 设置一致, 开发环境开启，生产环境关闭
              importLoaders: 0, // 指定在 CSS loader 处理前使用的 laoder 数量
            },
          },
        ],
      },
      {
        test: /\.less$/,
        use: [
          isDev ? 'style-loader' : MiniCssExtractPlugin.loader,
          {
            loader: 'css-loader',
            options: {
              modules: false,
              sourceMap: isDev,
              importLoaders: 1, // 需要先被 less-loader 处理，所以这里设置为 1
            },
          },
          {
            loader: 'less-loader',
            options: {
              sourceMap: isDev,
            },
          },
        ],
      },
      {
        test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10 * 1024,
              name: '[name].[contenthash:8].[ext]',
              outputPath: 'assets/images',
            },
          },
        ],
      },
      {
        test: /\.(ttf|woff|woff2|eot|otf)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              name: '[name].[contenthash:8].[ext]',
              outputPath: 'assets/fonts',
            },
          },
        ],
      },
      {
        test: /\.(tsx?|js)$/,
        loader: 'babel-loader',
        options: { cacheDirectory: true },
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    // 静态资源处理
    new CopyWebpackPlugin({
      patterns: [
        {
          context: resolve(PROJECT_PATH, './public'),
          from: '*',
          to: resolve(PROJECT_PATH, './dist'),
          toType: 'dir',
          globOptions: {
            dot: true,
            gitignore: true,
            ignore: ['**/index.html'],
          },
        },
      ],
    }),
    new HtmlWebpackPlugin({
      template: resolve(PROJECT_PATH, './public/index.html'),
      filename: 'index.html',
      cache: false, // 特别重要：防止之后使用v6版本 copy-webpack-plugin 时代码修改一刷新页面为空问题。
      minify: isDev
        ? false
        : {
            removeAttributeQuotes: true,
            collapseWhitespace: true,
            removeComments: true,
            collapseBooleanAttributes: true,
            collapseInlineTagWhitespace: true,
            removeRedundantAttributes: true,
            removeScriptTypeAttributes: true,
            removeStyleLinkTypeAttributes: true,
            minifyCSS: true,
            minifyJS: true,
            minifyURLs: true,
            useShortDoctype: true,
          },
    }),
    // 显示编译进度
    new WebpackBar({
      name: isDev ? 'run' : 'build',
      color: isDev ? '#00b2a9' : '#ee6139',
    }),
    // 编译时进行 typescript 类型检测
    new ForkTsCheckerWebpackPlugin({
      typescript: {
        configFile: resolve(PROJECT_PATH, './tsconfig.json'),
      },
    }),
  ],
};
```

- 在 config 文件夹下新建 webpack.dev.js 文件，配置开发环境

```js
const { merge } = require('webpack-merge');
const webpack = require('webpack');
const common = require('./webpack.common');
const { SERVER_HOST, SERVER_PORT } = require('./constant');

module.exports = merge(common, {
  mode: 'development',
  devtool: 'cheap-module-source-map',
  devServer: {
    host: SERVER_HOST, // 指定 host，不设置的话默认是 localhost
    port: SERVER_PORT, // 指定端口，默认是8080
    // stats: 'errors-only', // 终端仅打印 error
    // clientLogLevel: 'silent', // 日志等级
    compress: true, // 是否启用 gzip 压缩
    open: true, // 打开默认浏览器
    hot: true, // 热更新
  },
  plugins: [new webpack.HotModuleReplacementPlugin()],
});
```

- 在 config 文件夹下新建 webpack.prod.js 文件，配置开发环境

```js
const { merge } = require('webpack-merge');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const PurgeCSSPlugin = require('purgecss-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const glob = require('glob');
const { resolve } = require('path');
const common = require('./webpack.common');
const { PROJECT_PATH } = require('./constant');

module.exports = merge(common, {
  mode: 'production',
  devtool: false,
  plugins: [
    // 清除dist目录
    new CleanWebpackPlugin(),
    // 抽离 css 单独打包并且去除无用 css 代码
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
      chunkFilename: 'css/[name].[contenthash:8].chunk.css',
    }),
    new PurgeCSSPlugin({
      paths: glob.sync(`${resolve(PROJECT_PATH, './src')}/**/*.{tsx,scss,less,css}`, { nodir: true }),
      whitelist: ['html', 'body'],
    }),
  ],
  // 把一些引用的第三方包也打成单独的 chunk
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        extractComments: false,
        terserOptions: {
          compress: { pure_funcs: ['console.log'] },
        },
      }),
      new CssMinimizerPlugin(),
    ],
    splitChunks: {
      chunks: 'all',
      minSize: 0,
    },
  },
});
```

## eslint

```
npx eslint --init
```

### prettier 和 eslint 冲突问题

- 安装插件 eslint-config-prettier，禁用和 prettier 起冲突的规则
  ```
  npm install eslint-config-prettier -D
  ```
- 在 .eslintrc.js 的 extends 中加入以下代码
  ```
  'prettier'
  ```

## 添加 index.html

新建`index.html`文件

```html
<!-- sourced from https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
    <title>Create React App</title>
  </head>

  <body>
    <div id="root"></div>
    <noscript> You need to enable JavaScript to run this app. </noscript>
    <script src="../dist/bundle.js"></script>
  </body>
</html>
```

## 添加 src/index.jsx、src/App.jsx

```js
// index.jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

if (module && module.hot) {
  module.hot.accept();
}

ReactDOM.render(<App />, document.querySelector('#root'));
```

```js
// App.jsx
import React from 'react';

function App() {
  return (
    <div style={{ textAlign: 'center' }}>
      <h1>hello world</h1>
    </div>
  );
}

export default App;
```

## 其他配置

.gitignore git 忽略文件配置

```
  /node_modules
  /build
  /dist

```

.prettierrc.js prettier 配置文件

```js
module.exports = {
  arrowParens: 'avoid', // 箭头函数只有一个参数的时候可以忽略括号
  bracketSpacing: true, // 括号内部不要出现空格
  endOfLine: 'lf', // 行结束符使用 Unix 格式
  jsxBracketSameLine: false, // 在jsx中把'>' 是否单独放一行
  jsxSingleQuote: false, // jsx 属性使用双引号
  printWidth: 140, // 行宽
  proseWrap: 'preserve', // 换行方式
  semi: true, // 句尾添加分号
  singleQuote: true, // 使用单引号
  tabWidth: 2, // 缩进
  useTabs: false, // 使用空格缩进
  trailingComma: 'es5', // 在对象或数组最后一个元素后面是否加逗号（在ES5中加尾逗号）
  bracketSpacing: true, // 在对象，数组括号与文字之间加空格
  semicolons: true, // 在语句末尾打印分号
};
```

.eslintignore 配置文件

```
/node_modules
/build
/dist
```
