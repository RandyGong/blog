---
title: JavaScript AST Practice
date: 2020-01-07 23:44:33
categories:
- Javascript
tags: 
- AST
---

## Introduction

Each programming language has its own AST. Understanding AST and being able to carry out some development will provide great convenience for our project development. Let's take you to find out.

## Introduction to JS AST

AST is also called abstract syntax tree. In short, it is to show the program in tree form.
Each language (HTML, CSS, JS, etc.) has its own AST, and there are many kinds of AST parsers.

The common AST parsers for JS are:

- acorn
- @babel/parser
- Typescript
- Uglify-js
- etc.

The ASTs parsed by different parsers are slightly different, but they are essentially the same.
This article will explain based on @Babel/parser.

A common code:
```JavaScript
import ajax from 'axios'
```

The transformed AST structure is as follows:

```JavaScript
{
    "type": "ImportDeclaration",
    "start": 0,
    "end": 24,
    "loc": {
      "start": {
        "line": 1,
        "column": 0
      },
      "end": {
        "line": 1,
        "column": 24
      }
    },
    "specifiers": [
      {
        "type": "ImportDefaultSpecifier",
        "start": 7,
        "end": 11,
        "loc": {
          "start": {
            "line": 1,
            "column": 7
          },
          "end": {
            "line": 1,
            "column": 11
          }
        },
        "local": {
          "type": "Identifier",
          "start": 7,
          "end": 11,
          "loc": {
            "start": {
              "line": 1,
              "column": 7
            },
            "end": {
              "line": 1,
              "column": 11
            },
            "identifierName": "ajax"
          },
          "name": "ajax"
        }
      }
    ],
    "importKind": "value",
    "source": {
      "type": "StringLiteral",
      "start": 17,
      "end": 24,
      "loc": {
        "start": {
          "line": 1,
          "column": 17
        },
        "end": {
          "line": 1,
          "column": 24
        }
      },
      "extra": {
        "rawValue": "axios",
        "raw": "'axios'"
      },
      "value": "axios"
    }
  }
```

![](https://app.yinxiang.com/shard/s57/res/6d9dda6a-4d4e-4322-a112-d040657c64ca/640.jpg)

## ImportDeclaration
The common ones are:

- VariableDeclaration：var x = 'init'

- FunctionDeclaration：function func(){}

- ExportNamedDeclaration：export function exp(){}

- IfStatement：if(1>0){}

- WhileStatement：while(true){}

- ForStatement：for(;;){}

Since it is an introduction expression, it is naturally divided into two parts: specifiers on the left and source on the right.

## specifiers
The specifiers node will have a list to hold the specifiers.
If only one variable is declared on the left, an importdefaultspecifier is given.
If there are multiple declarations on the left, there will be a list of importspecifiers.

```JavaScript
import {a,b,c} from 'x'
```

The declaration of variables should be unique, and identifier is responsible for this.

## source
The source contains a string node, stringliteral, which corresponds to the location of the reference resource. It is Axios in the example.

## How is ast converted?

Take Babel as an example:
```JavaScript
const parser = require('@babel/parser')
let codeString = `
import ajax from 'axios'
`;


let file = parser.parse(codeString,{
    sourceType: "module"
})
console.dir(file.program.body)
```

Run it in node, and you can print out AST.

## Practice Scenarios

- Transformation of Babel to ES6 grammar
- Webpack collection of dependencies
- Compression of code by uglify JS
- On demand loading Babel plugin of component library
- etc.

In order to better understand AST, we define a scene and then practice it.
Scenario: convert import to require, similar to Babel
Objective: to transform the statement by AST

Turn
```JavaScript
import ajax from 'axios'
```
into
```JavaScript
var ajax = require('axios')
```

To achieve this effect, we first need to write a Babel plugin. Code first:
The code of babelPlugin.js is as follows:

```JavaScript
const t = require('@babel/types');

module.exports = function babelPlugin(babel) {
  function RequireTranslator(path){
    var node = path.node
    var specifiers = node.specifiers

    // get variable name
    var varName = specifiers[0].local.name;
    // get resource path
    var source = t.StringLiteral(path.node.source.value)
    var local = t.identifier(varName)
    var callee = t.identifier('require')
    var varExpression = t.callExpression(callee,[source])
    var declarator = t.variableDeclarator(local, varExpression)
    // create new node
    var newNode = t.variableDeclaration("var", [declarator])
    // node replacement
    path.replaceWith(newNode)
  }

  return {
    visitor: {
      ImportDeclaration(path) {
        RequireTranslator.call(this,path)      
      }
    }
  };
};
```
Test code:
```JavaScript
const babel = require('@babel/core');
const babelPlugin = require('./babelPlugin')


let codeString = `
import ajax from 'axios'
`;
const plugins = [babelPlugin]
const {code} = babel.transform(codeString,{plugins:plugins});
console.dir(code)
```

Output:
```JavaScript
'var ajax = require("axios");'
```

Goal achieved!

## Lastly
The AST of JS provides us with various possible opportunities. We can customize a syntax, simplify the introduction of components on demand, and so on. At the same time, not only JS, CSS, HTML, SQL can do some interesting operations at the AST syntax level. This article is just a simple introduction. Write at the end: the front end is not just the UI, there are many things to play!