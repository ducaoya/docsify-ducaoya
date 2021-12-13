# 关于在 React 项目中使用 Tailwind CSS 的简单记录

## 关于 Tailwind CSS

Tailwind CSS 就是一个 CSS 框架，正如你所知道的 bootstrap，element ui，Antd，bulma。一样。将一些 css 样式封装好，用来加速我们开发的一个工具 。

但是它有与上述的框架有所不同，上述的框架更多的是一种可复用性的组件，例如一个按钮、一个表单、或者一个表格等等，他们封装了大量的常用 UI 组件。但是 Tailwind CSS 与他们有本质的不同，它并没有封装某个具体的组件，而是封装了大量可复用的 css 样式。antd 等框架的优势是在你需要某种实现某一个 UI 组件的时候，可以拿来即用，可以快速高效的交付一个项目，但是也有其缺点，他们都封装都带的样式，而当你想要自定义一些个性化的样式的时候，就会发现这种操作比较困难，自由度很低，强行去修改它的样式会很难看，而且这些操作也会不优雅。

而 Tailwind CSS 则解决了这样的弊端，他自由度极高，可以随心所欲的实现我们想要的效果。我们仅需将按我们的需求去引入一个个基本的样式，也可以加入一些自己的样式。更值得一提的是，虽然他的自由度极高，但是他的设计却可以保证下项目的中组件的风格一致性，并且其自带的一些专业的颜色体系，也是非常值得学习和使用的。同样的也有自己的缺点，首先是他的引入方式，是通过 class 去进行引入，这将导致某些组件或者元素的 class 非常的长，html 文件里就显得很杂乱；另外还有他的学习成本也比较高（当然这是每一种框架一定会出现的），初学者使用的时候，需要不断的去查看文档（当然也可以使用 vscode 插件，学会其语义化规则还是挺方便的）；最后，因为他是一些纯 CSS，缺少与 JavaScript 的结合，所以很多需要配合 JavaScript 实现的组件，如下菜单等等，均没有办法通过纯粹的 Tailwind CSS 去实现这样的效果。

## 安装和配置

1. 创建 react 项目

```
npx create-react-app tailwindcss_demo --template typyscript
```

2. 初始化 tailwind CSS

- 安装 Tailwind 以及其它依赖项:

```
npm install -D tailwindcss@npm:@tailwindcss/postcss7-compat postcss@^7 autoprefixer@^9
```

> 据说 Create React App 尚未支持 PostCSS 8，所以需要安装 Tailwind CSS v2.0 PostCSS 7 兼容性版本。

- 安装和配置 CRACO

```
npm install @craco/craco
```

安装完毕后，更新 package.json 文件中的 scripts，将 eject 以外的所有脚本都用 craco 代替 react-scripts。

```json
{
  // ...
  "scripts": {
    "start": "craco start",
    "build": "craco build",
    "test": "craco test",
    "eject": "react-scripts eject"
  }
}
```

接下来，在项目根部创建一个 craco.config.js，并添加 tailwindcss 和 autoprefixer 作为 PostCSS 插件。

```js
// craco.config.js
module.exports = {
  style: {
    postcss: {
      plugins: [require('tailwindcss'), require('autoprefixer')],
    },
  },
}
```

3. 创建配置文件

```
npx tailwindcss-cli@latest init
```

这将会在您的项目根目录创建一个最小化的 tailwind.config.js 文件：

```js
// tailwind.config.js
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

4. 配置 Tailwind 来移除生产环境下没有使用到的样式声明

   在您的 tailwind.config.js 文件中，配置 purge 选项指定所有的 components 文件，使得 Tailwind 可以在生产构建中对未使用的样式进行摇树优化。

```js
// tailwind.config.js
module.exports = {
  purge: ['./src/**/*.{js,jsx,ts,tsx}', './public/index.html'],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

5. 在 CSS 中引入 Tailwind

   打开 Create React App 默认为您生成的 ./src/index.css 文件 并使用 @tailwind 指令来包含 Tailwind 的 base、 components 和 utilities 样式，来替换掉原来的文件内容。

```
/* ./src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Tailwind 会在构建时将这些指令转换成所有基于您配置的设计系统生成的样式文件。  
最后，确保您的 CSS 文件被导入到您的 ./src/index.js 文件中。

```js
// src/index.js
import React from 'react'
import ReactDOM from 'react-dom'
import './index.css'
import App from './App'
import reportWebVitals from './reportWebVitals'

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
)

// ...
```

6. 启动项目

   使用 `yarn start` 不太行，识别不到 craco 指令，必须使用 `npm run start`

```
npm run start
```

## 相关资料

- [tailwind css 中文文档](https://www.tailwindcss.cn/docs/installation)（据说并非官方在维护此文档）
- [tailwind css 官方文档](https://tailwindcss.com/docs/installation)
- [项目 demo 可前往 github](https://github.com/ducaoya/demos)
