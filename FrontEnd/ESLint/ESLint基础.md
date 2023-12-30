# ESLint基础

## 简述
eslint属于QA（Quality Assurance）工具，是一个ECMAScript/JavaScript语法规则和代码风格的检查工具，可以用来保证写出语法正确、风格统一的代码。
eslint完全可配置，它的目标是提供一个插件化的JavaScript代码检测工具。这意味着您可以关闭每个规则，只能使用基础语言验证，或者混合并匹配捆绑的规则和自定义规则，使eslint完美的适用于当前项目。
优势

- 保证团队协作开发时，编写出来的代码风格保持一致。
   - 函数名和括号之间要有一个空格
   - JS中的字符串，统一使用单引号表示
   - 一行代码结束加不加分号
   - 不允许出现大于2个的连续空行
   - import 必须放到文件的最上面
   - 文件结尾要留一行空行
   - 对象的末尾不要有多余的逗号
## 创建Eslint项目
```powershell
npm init -y 
npm init @eslint/config

# 找问题
npx eslint ./
```
**改正错误的方式**
有三种方法来修正错误：

- 手动修正: 手动修改
- 命令修正：npm run lint
- 插件修正: 配合vscode 中的eslint插件
## ESLint配置
### 配置规则
ESLint 有大量的内置规则，你可以通过插件添加更多的规则。你也可以通过配置注释或配置文件来修改你的项目使用哪些规则。要改变一个规则的设置，你必须把规则的 ID 设置为这些值之一。

- "off" 或 0 - 关闭规则
- "warn" 或 1 - 启用并视作警告（不影响退出）。
- "error" 或 2 - 启用并视作错误（触发时退出代码为 1）
```powershell
{
    "rules": {
        "eqeqeq": "off",
        "curly": "error",
        "quotes": ["error", "double"]
    }
}
```
### 禁用规则
#### 禁用所有规则
```javascript
/* eslint-disable */

alert('foo');

/* eslint-enable */
```
#### 禁用特定规则
```javascript
/* eslint-disable no-alert, no-console */

alert('foo');
console.log('bar');

/* eslint-enable no-alert, no-console */
```
#### 禁用特定行规则
```javascript
alert('foo'); // eslint-disable-line

// eslint-disable-next-line
alert('foo');

/* eslint-disable-next-line */
alert('foo');

alert('foo'); /* eslint-disable-line */
```
#### 要禁用某一特定行的特定规则
```javascript
alert('foo'); // eslint-disable-line no-alert

// eslint-disable-next-line no-alert
alert('foo');

alert('foo'); /* eslint-disable-line no-alert */

/* eslint-disable-next-line no-alert */
alert('foo');
```
#### 要禁用一个特定行的多个规则
```javascript
alert('foo'); // eslint-disable-line no-alert, quotes, semi

// eslint-disable-next-line no-alert, quotes, semi
alert('foo');

alert('foo'); /* eslint-disable-line no-alert, quotes, semi */

/* eslint-disable-next-line no-alert, quotes, semi */
alert('foo');

/* eslint-disable-next-line
  no-alert,
  quotes,
  semi
*/
alert('foo');
```
案例
```javascript
module.exports = {
    "env": {
        "browser": true,
        "es2021": true,
        "node": true
    },
    "extends": [
        "eslint:recommended",
        "plugin:@typescript-eslint/recommended",
        "plugin:vue/vue3-essential"
    ],
    "overrides": [
        {
            "env": {
                "node": true
            },
            "files": [
                ".eslintrc.{js,cjs}"
            ],
            "parserOptions": {
                "sourceType": "script"
            }
        }
    ],
    "parserOptions": {
        "ecmaVersion": "latest",
        "parser": "@typescript-eslint/parser",
        "sourceType": "module"
    },
    "plugins": [
        "@typescript-eslint",
        "vue"
    ],
    "rules": {
        "eqeqeq": "error",
        "curly": "error",
        "quotes":["error","double"]
    }
}

```

.eslintignore
```javascript
src/index.js
```
