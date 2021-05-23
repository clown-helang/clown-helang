---
title: JS类SQL语句解析引擎
catalog: true
date: 2021-05-23 16:31:53
subtitle: JS解析引擎概念应用
header-img: "/img/article_header/article_header.png"
tags:
- Javascript
---

## 前言
思考一下，如果我们现在有下面一段纯文本：
```
(创建时间 属于 "当天") AND (处置状态 属于 "待处置,处置中,已处置" AND 处置人 是否为空 "否")
```
现在需要解析成下面的一个树形结构：
```
{
  'relation': 'AND',
  'statements': [
    {
      'identifier': '创建时间',
      'operator': '属于',
      'value': '当天'
    },
    {
      'relation': 'AND',
      'statements': [
        {
          'identifier': '处置状态',
          'operator': '属于',
          'value': [
            '待处置',
            '处置中',
            '已处置'
          ]
        },
        {
          'identifier': '处置人',
          'operator': '是否为空',
          'value': '否'
        }
      ]
    }
  ]
}
```
我们该如何做呢？

## 功能分析
将一段纯文本解析为一个具有层级关系的语法树，从结果来看，这个过程与`JS`的代码解析执行过程是极其相似的，势必需要经过词法分析、语法分析和语义分析这三个阶段。这里我们可以不考虑最终生成的语法树是否符合语义。只要语法树的层级关系正确即可。

## 功能设计

### 词法分析
从左至右依次遍历每个字符，以空格作为分隔符，将整个纯文本分割成若干个`token`，最终生成内容如下所示：
```
// tokens
['(', '创建时间', '属于', '"当天"', ')', 'AND', '(', '处置状态', '属于', '"待处置,处置中,已处置"', 'AND', '处置人', '是否为空', '"否"', ')'];
```
生成的过程中可以对于内容进行基本`词法校验`,例如：是否缺少`"`、`(`、`)`、`'`等

### 语法分析
经过词法分析后我们可以拿到一组`tokens`,那么我们该如何将它们解析成具体的语法树呢？这个时候我们就需要对每一个`token`的含义进行分析了。举个小栗子：`创建时间 属于 "当天"`。整个语句中可以划分为三个角色：`关键字`、`操作符`、`操作值`。有了这个概念之后，那么其他部分也就可以同样进行角色的划分，那么我们就可以得出下面这些角色：

- `关键字`：`创建时间`、`处置状态`、`处置人`
- `操作符`：`属于`、`是否为空`
- `操作值`：`当天`、`"待处置,处置中,已处置"`、`否`
- `关系符`：`AND`、`OR`
- `结束符`：`(`、`)`

有了这些角色之后我们就可以进行具体的语法分析了。

第一步：遍历`tokens`，解析最小表达式
```
[
    '(',
    {
      'identifier': '创建时间',
      'operator': '属于',
      'value': '当天'
    },
    ')',
    'AND',
    '(',
    {
      'identifier': '处置状态',
      'operator': '属于',
      'value': [
        '待处置',
        '处置中',
        '已处置'
      ]
    },
    'AND',
    {
      'identifier': '处置人',
      'operator': '是否为空',
      'value': '否'
    },
    ')'
]
```

第二步：生成逻辑组合最小表达式

```
[
    '(',
    {
      'identifier': '创建时间',
      'operator': '属于',
      'value': '当天'
    },
    ')',
    'AND',
    '(',
    {
      'relation': 'AND',
      'statements': [
        {
          'identifier': '处置状态',
          'operator': '属于',
          'value': [
            '待处置',
            '处置中',
            '已处置'
          ]
        },
        {
          'identifier': '处置人',
          'operator': '是否为空',
          'value': '否'
        }
      ]
    },
    ')'
]
```

第三步：去括号

```
[
    {
      'identifier': '创建时间',
      'operator': '属于',
      'value': '当天'
    },
    'AND',
    {
      'relation': 'AND',
      'statements': [
        {
          'identifier': '处置状态',
          'operator': '属于',
          'value': [
            '待处置',
            '处置中',
            '已处置'
          ]
        },
        {
          'identifier': '处置人',
          'operator': '是否为空',
          'value': '否'
        }
      ]
    }
]
```

第四步：递归前三步，直到`tokens`长度为1时，返回`tokens[0]`。

```
[
    {
      'relation': 'AND',
      'statements': [
        {
          'identifier': '创建时间',
          'operator': '属于',
          'value': '当天'
        },
        {
          'relation': 'AND',
          'statements': [
            {
              'identifier': '处置状态',
              'operator': '属于',
              'value': [
                '待处置',
                '处置中',
                '已处置'
              ]
            },
            {
              'identifier': '处置人',
              'operator': '是否为空',
              'value': '否'
            }
          ]
        }
      ]
    }
]
```
至此，我们变可以拿到我们想要的语法树（AST）了,在解析最小表达式的过程中，我也可以进行基础的语法校验，比如是否符合: `关键字` `操作符` `值` 的基本格式等。

## 功能实现

### 解析引擎
```javascript
// parsing-engine.js
/**
 * ParsingEngine 解析引擎
 */
import { isValue, isNumber, isString, isObject, formatValue } from './utils';
const LEFT_BRACKET = '(';
const RIGHT_BRACKET = ')';
const END_SYMBOLS = [LEFT_BRACKET, RIGHT_BRACKET, ' '];

class ParsingEngine {
  constructor({relations, operations, identifiers}) {
    this.relations = relations || ['AND', 'OR'];
    this.operations = operations || [];
    this.tokens = [];
    this.identifiers = identifiers;
  }

  /**
  * 词法分析
  * @param {String} str
  */
  lex(str) {
    this.lexCheck(str);

    let tokens = [], token = '';
    for (let i = 0; i < str.length; i++) {
      const char = str.charAt(i);
      if (!END_SYMBOLS.includes(char)) {
        token += char;
      } else {
        token && (tokens.push(token), token = '');
        tokens.push(char);
      }
    }
    tokens = tokens.filter(v => v !== ' ');
    this.tokens = tokens;
  }

  /**
   * 词法校验
   * @param {String} str
   */
  lexCheck(str) {
    const leftBracketLength = str.split(LEFT_BRACKET).length - 1;
    const rightBracketLength = str.split(RIGHT_BRACKET).length - 1;
    const differenceValue = leftBracketLength - rightBracketLength;
    if (differenceValue !== 0) {
      throw new Error(`语法错误: 缺少 "${differenceValue > 0 ? RIGHT_BRACKET : LEFT_BRACKET}"`);
    }

    const doubleQuoteLength = str.split('"').length - 1;
    if (doubleQuoteLength % 2) {
      throw new Error('语法错误: 缺少 \'"\'');
    }

    const singleQuoteLength =  str.split('\'').length - 1;
    if (singleQuoteLength % 2) {
      throw new Error('语法错误: 缺少 "\'"');
    }
  }

  /**
  * 语法分析
  */
  parser() {
    const { tokens, relations, operations } = this;

    /**
    * 1. 解析最小表达式
    */
    tokens.forEach((token, index) => {
      if (!this.grammarCheck(token, tokens[index + 1])) {
        throw new Error(`语法错误："${token} ${tokens[index + 1]}"`);
      }
      if (operations.includes(token)) {
        const identifier = tokens[index - 1];

        let value = tokens[index + 1];
        if (isNumber(value)) value = (+value);
        if (isString(value)) value = formatValue(value);
        const statement = { identifier, operator: token, value };

        tokens[index - 1] = statement;
        tokens.splice(index, 2);
      }
    });
    /**
    * 2. 逻辑组合最小表达式
    */
    tokens.forEach((item, index) => {
      if (relations.includes(item)) {
        const left = tokens[index - 1];
        const right = tokens[index + 1];

        if (isObject(left) && isObject(right)) {
          if (left.statements && left.relation === item) {
            left.statements.push(right);
          } else {
            const statement = {
              relation: item,
              statements: [left, right]
            };
            tokens[index - 1] = statement;
          }

          tokens.splice(index, 2);
        }
      }
    });
    /**
    * 3. 去括号
    */
    tokens.forEach((item, index) => {
      if (isObject(item)) {
        const isLeftBracket = tokens[index - 1] === LEFT_BRACKET;
        const isRightBracket = tokens[index + 1] === RIGHT_BRACKET;
        const isBracket = isLeftBracket && isRightBracket;

        if (isBracket) {
          tokens[index - 1] = item;
          tokens.splice(index, 2);
        }
      }
    });
    /**
    * 4. 递归
    */
    if (tokens.length > 1) {
      this.tokens = tokens;
      return this.parser();
    }
    return tokens[0];
  }

  /**
   * 语法校验
   */
  grammarCheck(prev, next) {
    /**
     * eg:
     * - ( 创建时间
     * - ( (
     */
    const isBracketBegin = (LEFT_BRACKET === prev) && (this.identifiers.includes(next) || LEFT_BRACKET === next || isObject(next));
    if (isBracketBegin) return true;
    /**
     * eg:
     * - 创建时间 属于
     */
    const isStatementLeft = this.identifiers.includes(prev) && this.operations.includes(next);
    if (isStatementLeft) return true;
    /**
     * eg:
     * - 属于 "当天"
     */
    const isStatementRight = this.operations.includes(prev) && isValue(next);
    if (isStatementRight) return true;
    /**
     * eg:
     * - { identifier: '剩余天数', operator: '>=', value: 3  } AND
     * - ) OR
     */
    const isRelationLeft = (RIGHT_BRACKET === prev || isObject(prev)) && this.relations.includes(next);
    if (isRelationLeft) return true;
    /**
     * eg:
     * - AND (
     * - AND { identifier: '剩余天数', operator: '>=', value: 3  }
     */
    const isRelationRight = this.relations.includes(prev) && (LEFT_BRACKET === next || isObject(next));
    if (isRelationRight) return true;
    /**
     * eg:
     * -  当天 )
     * - { identifier: '剩余天数', operator: '>=', value: 3  } )
     * - ))
     * - )
     */
    const isBracketEnd = ((RIGHT_BRACKET === prev) && next === undefined) || ((isValue(prev) || isObject(prev) || RIGHT_BRACKET === prev) && (RIGHT_BRACKET === next));
    if (isBracketEnd) return true;

    /**
     * eg:
     * - { identifier: '剩余天数', operator: '>=', value: 3  }
     */
    const isObjectEnd = isObject(prev) && next === undefined;
    if (isObjectEnd) return true;

    return false;
  }

  run(str) {
    this.lex(str);
    return this.parser();
  }
}

export default ParsingEngine;

```

### 工具方法
```javascript
// utils.js
export function isNumber(value) {
  return Number(value).toString() !== 'NaN';
}

export function isString(value) {
  if (typeof value !== 'string') return false;
  const firstChar = value.charAt(0);
  const lastChar = value.charAt(value.length - 1);
  return (firstChar === '"' && lastChar === '"') || (firstChar === '\'' && lastChar === '\'');
}

export function isObject(value) {
  return Object.prototype.toString.call(value) === '[object Object]';
}

export function isValue(value) {
  return isNumber(value) || isString(value);
}

export function formatValue(value) {
  value = value.replace(/"|'/g, '');
  if (value.indexOf(',') > -1) {
    value = value.split(',');
  }
  return value;
}

```
