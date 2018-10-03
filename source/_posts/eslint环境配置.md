---
title: eslint环境配置
date: 2018-09-10 17:02:47
tags: eslint 代码规范
---

今天教师节，首先祝愿各位老师节日快乐！

eslint是一个静态代码检查工具，一般一个中大型项目会有多人共同开发及维护，为了保证所有代码的可读性可维护性同时也由于很多公司的code review机制并不完善，在项目中使用eslint可以实现开发人员统一代码规范。下面就简单得聊一下如何在项目中优雅得使用eslint

### 1.全局安装eslint 
```bash
npm i eslint -g
```

### 2.初始化eslint配置
```bash
eslint --init
```
这一步会有很多选项，按照提示跟你项目的实际需要选择就行，最终会生成一个.eslintrc配置文件，扩展名可以是json js等等，这个随意我选的是js
这个配置文件就包含刚刚在命令行中选择的内容

### 3.安装eslint项目依赖
```bash
npm i eslint -D // 这个是最基本配置
npm i eslint-plugin-react -D // 如果你的项目使用了react作为框架，这个eslint插件必不可少
npm i eslint-plugin-node -D // 如果你的项目还需要支持node环境，比如使用process对象而不报错
```
目前市面上已经有了公开的eslint配置文件，公认最好的是airbnb开源出来的eslint规范, 但是内部规范比较严格，可能并不适合所有团队，你可以在eslintrc文件中进行自定义
```bash
npm i eslint-config-airbnb -D
```
下面是我自己配置的eslint(注：是一个以react为view层框架的election项目)
```javascript
module.exports = {
  // 环境定义了预定义的全局变量。
	"env": {
		"browser": true, // 表示支持浏览器环境
		"commonjs": true, // 表示需要支持commonJS规范
		"es6": true, // 表示需要支持es6语法
		"node": true // 表示支持node环境
	},
	"extends": "./node_modules/eslint-config-airbnb/.eslintrc", // 表示默认airbnb的eslint配置
  // 某些es7/8语法eslint并不能识别就会报一个Parsing error错误，这个时候就需要安装一个解析器 npm i babel-eslint -D
  // 默认使用 Espree
	"parser": "babel-eslint",
  // parserOptions 决定了Eslint认为哪些是错误
	"parserOptions": {
		"ecmaFeatures": {
			"jsx": true
		},
		"ecmaVersion": 2018,
		"sourceType": "module"
	},
  // 使用的插件
	"plugins": [
		"react"
	],
  /**
   * 自定义规则
   * 1.规则名：0 禁止 1警告 2错误
   * 2.规则名：["error": { options }]
   * 具体可以到eslint官网查询
   * 如果是插件的话需要到插件所在的npm官网或者github上去查，比如react/forbid-prop-types就需要到npm官网查询eslint-plugin-react
   */
	"rules": {
		"indent": ["error", 2, { "SwitchCase": 1 }],
		"quotes": [
			"error",
			"single"
		],
		"space-before-function-paren": ["error", {
			"anonymous": "always",
			"named": "always",
			"asyncArrow": "always"
		}],
		"object-curly-newline": ["error", {
			"ObjectExpression": "always",
			"ObjectPattern": { "multiline": true }
		}],
		"linebreak-style": 0,
		"no-plusplus": 0,
		"consistent-return": 0,
		"no-param-reassign": 0,
		"object-curly-newline": 0,
		"comma-dangle": ["error", "never"],
		"semi": 0,
		"radix": 0,
		"max-len": ["error", {
			"code": 200,
			"ignoreComments": true,
			"ignoreUrls": true,
			"ignoreStrings": true,
			"ignoreTemplateLiterals": true,
			"ignoreRegExpLiterals": true 
		}],
		"no-console": 0,
		"no-var": 2,
		"no-trailing-spaces": 2,
		"no-unused-vars": [2, {
			"vars": "all",
			"args": "after-used"
		}],
		"no-underscore-dangle": 0,
		"no-lone-blocks": 2,
		"no-class-assign": 2,
		"no-cond-assign": 2,
		"no-delete-var": 2,
		"no-dupe-keys": 2,
		"no-duplicate-case": 2,
		"no-dupe-args": 2,
		"no-empty": 2,
		"no-func-assign": 2,
		"no-invalid-this": 2,
		"no-redeclare": 2,
		"no-spaced-func": 2,
		"no-this-before-super": 0,
		"no-undef": 2,
		"no-use-before-define": 1,
		"camelcase": 0,
		"jsx-quotes": [2, "prefer-double"],
		"react/jsx-boolean-value": 1,
		"react/jsx-closing-bracket-location": 2,
		"react/jsx-curly-spacing": [2, {
			"when": "never",
			"children": true
		}],
		"react/forbid-prop-types": 0,
		"react/jsx-sort-props": 0,
		"react/jsx-uses-vars": 2,
		"react/jsx-uses-react": 1,
		"react/no-danger": 1,
		"react/no-did-mount-set-state": 2,
		"react/no-did-update-set-state": 2,
		"react/no-direct-mutation-state": 2,
		"react/no-multi-comp": 2,
		"react/no-set-state": 0,
		"react/no-unknown-property": 2,
		"react/prefer-es6-class": 2,
		"react/prop-types": 2,
		"react/react-in-jsx-scope": 2,
		"react/self-closing-comp": 2,
		"react/sort-comp": 2,
		"no-extra-boolean-cast": 2,
		"react/no-array-index-key": 2,
		"react/no-deprecated": 2,
		"react/jsx-equals-spacing": 2,
		"react/jsx-filename-extension": [1, { "extensions": [".js", ".jsx"] }],
		"react/jsx-closing-tag-location": 0,
		"react/jsx-wrap-multilines": [1, {
			"declaration": "parens",
			"assignment": "parens",
			"return": "parens",
			"arrow": "parens",
			"condition": "ignore",
			"logical": "ignore",
			"prop": "ignore" 
		}],
		"react/prefer-stateless-function": 0,
		"react/destructuring-assignment": 0,
		"react/no-access-state-in-setstate": 0,
		"react/default-props-match-prop-types": 0,
		"react/require-default-props": 0,
		"react/jsx-one-expression-per-line": 0,
		"class-methods-use-this": 0,
		"no-unreachable": 1,
		"no-mixed-spaces-and-tabs": 0, //禁止混用tab和空格
		"prefer-arrow-callback": 0, //比较喜欢箭头回调
		"arrow-parens": 0, //箭头函数用小括号括起来
		"arrow-spacing": 0 //=>的前/后括号
	}
};
```
eslint是完全可配置，所有的规则都可以自定义是否启用，启用到什么程度
甚至可以在代码文件直接以注释的形式定义

```javascript
/* eslint-disable */

// Disables all rules between comments
alert('foo');

/* eslint-enable */

临时在一段代码中取消个别规则的检查（如no-alert, no-console）：

/* eslint-disable no-alert, no-console */

// Disables no-alert and no-console warnings between comments
alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */

在整个文件中取消eslint检查：

/* eslint-disable */

// Disables all rules for the rest of the file
alert('foo');

在整个文件中禁用某一项eslint规则的检查：

/* eslint-disable no-alert */

// Disables no-alert for the rest of the file
alert('foo');

针对某一行禁用eslint检查：

alert('foo'); // eslint-disable-line

// eslint-disable-next-line
alert('foo');

针对某一行的某一具体规则禁用eslint检查：

alert('foo'); // eslint-disable-line no-alert

// eslint-disable-next-line no-alert
alert('foo');

针对某一行禁用多项具体规则的检查：

alert('foo'); // eslint-disable-line no-alert, quotes, semi

// eslint-disable-next-line no-alert, quotes, semi
alert('foo');
```
[eslint中文官网](http://eslint.cn/)
[eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react)
[eslint-plugin-import](https://www.npmjs.com/package/eslint-plugin-import)
[eslint-plugin-jsx-a11y](https://www.npmjs.com/package/eslint-plugin-jsx-a11y)