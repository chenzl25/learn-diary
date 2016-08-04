# Pug-analysis

Pug工作原理:

1. pug-lexer
2. pug-parser
3. pug-load
4. pug-filters
5. pug-link
6. pug-code-gen

通过下面3个文件来阐述上面的6个步骤：

*./index.jade*
```pug
//- index.jade
doctype html
html
  include ./includes/head.jade
  body
    h1 My Site
    p Welcome to my super lame site.
    include ./includes/foot.jade
```

*./includes/head.jade*
```pug
//- includes/head.jade
head
  title My Site
  script(src='/javascripts/jquery.js')
  script(src='/javascripts/app.js')
```
*./includes/foot.jade*
```pug
//- includes/foot.jade
#footer
  p Copyright (c) foobar
```

## pug-lexer

通过将输入的字符串转化成一个个的token

index.jade的tokens
```json
[ { type: 'comment',
    line: 1,
    col: 1,
    val: ' index.jade',
    buffer: false },
  { type: 'newline', line: 2, col: 1 },
  { type: 'doctype', line: 2, col: 1, val: 'html' },
  { type: 'newline', line: 3, col: 1 },
  { type: 'tag', line: 3, col: 1, val: 'html' },
  { type: 'indent', line: 4, col: 1, val: 2 },
  { type: 'include', line: 4, col: 3 },
  { type: 'path', line: 4, col: 11, val: './includes/head.jade' },
  { type: 'newline', line: 5, col: 1 },
  { type: 'tag', line: 5, col: 3, val: 'body' },
  { type: 'indent', line: 6, col: 1, val: 4 },
  { type: 'tag', line: 6, col: 5, val: 'h1' },
  { type: 'text', line: 6, col: 8, val: 'My Site' },
  { type: 'newline', line: 7, col: 1 },
  { type: 'tag', line: 7, col: 5, val: 'p' },
  { type: 'text',
    line: 7,
    col: 7,
    val: 'Welcome to my super lame site.' },
  { type: 'newline', line: 8, col: 1 },
  { type: 'include', line: 8, col: 5 },
  { type: 'path', line: 8, col: 13, val: './includes/foot.jade' },
  { type: 'outdent', line: 8, col: 33 },
  { type: 'outdent', line: 8, col: 33 },
  { type: 'eos', line: 8, col: 33 } ]
```

缩进的语法格式通过token的形式`indent`, `outdent`给出。如果缩进不正确，在词法阶段就能检测出来。

### layout-syntax实现方法

tokenize的过程中遇到第一个缩进符就会判断是空格还是制表符。同一个文件一旦确定缩进符就会一直延续。
使用栈来记录缩进的历史深度，根据每次进来的缩进的深度判断是进栈还是出栈，出栈时会进行判断缩进是否正确。

*相关源码：*
```js
// outdent
if (indents < this.indentStack[0]) {
  while (this.indentStack[0] > indents) {
    if (this.indentStack[1] < indents) {
      this.error('INCONSISTENT_INDENTATION', 'Inconsistent indentation. Expecting either ' + this.indentStack[1] + ' or ' + this.indentStack[0] + ' spaces/tabs.');
    }
    this.colno = this.indentStack[1] + 1;
    this.tokens.push(this.tok('outdent'));
    this.indentStack.shift();
  }
// indent
} else if (indents && indents != this.indentStack[0]) {
  this.tokens.push(this.tok('indent', indents));
  this.colno = 1 + indents;
  this.indentStack.unshift(indents);
}
```

## pug-parser

通过将pug-lexer的tokens转化成AST

实现方法类似递归下降

类型:

- tag
- doctype
- mixin
- include
- filter
- comment
- text
- text-html
- dot
- each
- code
- yield
- id
- class
- interpolation

index.jade的AST
```
{
  "type": "Block",
  "nodes": [
    {
      "type": "Doctype",
      "val": "html",
      "line": 2,
      "filename": "test.jade"
    },
    {
      "type": "Tag",
      "name": "html",
      "selfClosing": false,
      "block": {
        "type": "Block",
        "nodes": [
          {
            "type": "Include",
            "file": {
              "type": "FileReference",
              "line": 4,
              "filename": "test.jade",
              "path": "./includes/head.jade"
            },
            "line": 4,
            "filename": "test.jade",
            "block": {
              "type": "Block",
              "nodes": [],
              "line": 4,
              "filename": "test.jade"
            }
          },
          {
            "type": "Tag",
            "name": "body",
            "selfClosing": false,
            "block": {
              "type": "Block",
              "nodes": [
                {
                  "type": "Tag",
                  "name": "h1",
                  "selfClosing": false,
                  "block": {
                    "type": "Block",
                    "nodes": [
                      {
                        "type": "Text",
                        "val": "My Site",
                        "line": 6,
                        "filename": "test.jade"
                      }
                    ],
                    "line": 6,
                    "filename": "test.jade"
                  },
                  "attrs": [],
                  "attributeBlocks": [],
                  "isInline": false,
                  "line": 6,
                  "filename": "test.jade"
                },
                {
                  "type": "Tag",
                  "name": "p",
                  "selfClosing": false,
                  "block": {
                    "type": "Block",
                    "nodes": [
                      {
                        "type": "Text",
                        "val": "Welcome to my super lame site.",
                        "line": 7,
                        "filename": "test.jade"
                      }
                    ],
                    "line": 7,
                    "filename": "test.jade"
                  },
                  "attrs": [],
                  "attributeBlocks": [],
                  "isInline": false,
                  "line": 7,
                  "filename": "test.jade"
                },
                {
                  "type": "Include",
                  "file": {
                    "type": "FileReference",
                    "line": 8,
                    "filename": "test.jade",
                    "path": "./includes/foot.jade"
                  },
                  "line": 8,
                  "filename": "test.jade",
                  "block": {
                    "type": "Block",
                    "nodes": [],
                    "line": 8,
                    "filename": "test.jade"
                  }
                }
              ],
              "line": 5,
              "filename": "test.jade"
            },
            "attrs": [],
            "attributeBlocks": [],
            "isInline": false,
            "line": 5,
            "filename": "test.jade"
          }
        ],
        "line": 3,
        "filename": "test.jade"
      },
      "attrs": [],
      "attributeBlocks": [],
      "isInline": false,
      "line": 3,
      "filename": "test.jade"
    }
  ],
  "line": 0,
  "filename": "test.jade"
}
```

## pug-load

将AST中Include，Extends类型的node中的路径path下的文件读取，并进行词法和语法分析，如果被解析的文件又包含了Include，Extends类型就递归调用pug-load的方法。
最后解析出来的AST会放置在Include，Extends类型的node的ast属性下。

## pug-linker

上一步的pug-linker得到的AST是需要整理的，例如：Include，Extends类型的node，这些节点需要删除，并将它们的AST和父AST连接起来。pug-linker干的就是这个事。

最终3个文件link后的结果
```json
{
  "type": "Block",
  "nodes": [
    {
      "type": "Doctype",
      "val": "html",
      "line": 2,
      "filename": "test.jade"
    },
    {
      "type": "Tag",
      "name": "html",
      "selfClosing": false,
      "block": {
        "type": "Block",
        "nodes": [
          {
            "type": "Block",
            "nodes": [
              {
                "type": "Tag",
                "name": "head",
                "selfClosing": false,
                "block": {
                  "type": "Block",
                  "nodes": [
                    {
                      "type": "Tag",
                      "name": "title",
                      "selfClosing": false,
                      "block": {
                        "type": "Block",
                        "nodes": [
                          {
                            "type": "Text",
                            "val": "My Site",
                            "line": 3,
                            "filename": "includes/head.jade"
                          }
                        ],
                        "line": 3,
                        "filename": "includes/head.jade"
                      },
                      "attrs": [],
                      "attributeBlocks": [],
                      "isInline": false,
                      "line": 3,
                      "filename": "includes/head.jade"
                    },
                    {
                      "type": "Tag",
                      "name": "script",
                      "selfClosing": false,
                      "block": {
                        "type": "Block",
                        "nodes": [],
                        "line": 4,
                        "filename": "includes/head.jade"
                      },
                      "attrs": [
                        {
                          "name": "src",
                          "val": "'/javascripts/jquery.js'",
                          "mustEscape": true
                        }
                      ],
                      "attributeBlocks": [],
                      "isInline": false,
                      "line": 4,
                      "filename": "includes/head.jade"
                    },
                    {
                      "type": "Tag",
                      "name": "script",
                      "selfClosing": false,
                      "block": {
                        "type": "Block",
                        "nodes": [],
                        "line": 5,
                        "filename": "includes/head.jade"
                      },
                      "attrs": [
                        {
                          "name": "src",
                          "val": "'/javascripts/app.js'",
                          "mustEscape": true
                        }
                      ],
                      "attributeBlocks": [],
                      "isInline": false,
                      "line": 5,
                      "filename": "includes/head.jade"
                    }
                  ],
                  "line": 2,
                  "filename": "includes/head.jade"
                },
                "attrs": [],
                "attributeBlocks": [],
                "isInline": false,
                "line": 2,
                "filename": "includes/head.jade"
              }
            ],
            "line": 0,
            "filename": "includes/head.jade",
            "declaredBlocks": {}
          },
          {
            "type": "Tag",
            "name": "body",
            "selfClosing": false,
            "block": {
              "type": "Block",
              "nodes": [
                {
                  "type": "Tag",
                  "name": "h1",
                  "selfClosing": false,
                  "block": {
                    "type": "Block",
                    "nodes": [
                      {
                        "type": "Text",
                        "val": "My Site",
                        "line": 6,
                        "filename": "test.jade"
                      }
                    ],
                    "line": 6,
                    "filename": "test.jade"
                  },
                  "attrs": [],
                  "attributeBlocks": [],
                  "isInline": false,
                  "line": 6,
                  "filename": "test.jade"
                },
                {
                  "type": "Tag",
                  "name": "p",
                  "selfClosing": false,
                  "block": {
                    "type": "Block",
                    "nodes": [
                      {
                        "type": "Text",
                        "val": "Welcome to my super lame site.",
                        "line": 7,
                        "filename": "test.jade"
                      }
                    ],
                    "line": 7,
                    "filename": "test.jade"
                  },
                  "attrs": [],
                  "attributeBlocks": [],
                  "isInline": false,
                  "line": 7,
                  "filename": "test.jade"
                },
                {
                  "type": "Block",
                  "nodes": [
                    {
                      "type": "Tag",
                      "name": "div",
                      "selfClosing": false,
                      "block": {
                        "type": "Block",
                        "nodes": [
                          {
                            "type": "Tag",
                            "name": "p",
                            "selfClosing": false,
                            "block": {
                              "type": "Block",
                              "nodes": [
                                {
                                  "type": "Text",
                                  "val": "Copyright (c) foobar",
                                  "line": 3,
                                  "filename": "includes/foot.jade"
                                }
                              ],
                              "line": 3,
                              "filename": "includes/foot.jade"
                            },
                            "attrs": [],
                            "attributeBlocks": [],
                            "isInline": false,
                            "line": 3,
                            "filename": "includes/foot.jade"
                          }
                        ],
                        "line": 2,
                        "filename": "includes/foot.jade"
                      },
                      "attrs": [
                        {
                          "name": "id",
                          "val": "'footer'",
                          "mustEscape": false
                        }
                      ],
                      "attributeBlocks": [],
                      "isInline": false,
                      "line": 2,
                      "filename": "includes/foot.jade"
                    }
                  ],
                  "line": 0,
                  "filename": "includes/foot.jade",
                  "declaredBlocks": {}
                }
              ],
              "line": 5,
              "filename": "test.jade"
            },
            "attrs": [],
            "attributeBlocks": [],
            "isInline": false,
            "line": 5,
            "filename": "test.jade"
          }
        ],
        "line": 3,
        "filename": "test.jade"
      },
      "attrs": [],
      "attributeBlocks": [],
      "isInline": false,
      "line": 3,
      "filename": "test.jade"
    }
  ],
  "line": 0,
  "filename": "test.jade",
  "declaredBlocks": {}
}
```

## pug-code-gen

代码生成将AST转化为JavaScript中Function函数的Body。和ejs是一个套路。

而这个Function(*我们叫它render吧*)就是`render = require('jade').compile(str)`得到的一个函数。

最后`html = render(options)`。

下面代码是最终得到的render函数(*修改过*)

```js
function template(locals) {
  var pug_html = "", pug_mixins = {}, 
      pug_interp;var pug_debug_filename, 
      pug_debug_line;
  try {
    pug_html = pug_html + "<!DOCTYPE html>";
    pug_html = pug_html + "<html>";
    pug_html = pug_html + "<head>";
    pug_html = pug_html + "<title>";
    pug_html = pug_html + "My Site</title>";
    pug_html = pug_html + "<script src=\"/javascripts/jquery.js\"></script>";
    pug_html = pug_html + "<script src=\"/javascripts/app.js\"></script></head>";
    pug_html = pug_html + "<body>";
    pug_html = pug_html + "<h1>";
    pug_html = pug_html + "My Site</h1>";
    pug_html = pug_html + "<p>";
    pug_html = pug_html + "Welcome to my super lame site.</p>";
    pug_html = pug_html + "<div id=\"footer\">";
    pug_html = pug_html + "<p>";
    pug_html = pug_html + "Copyright (c) foobar</p></div></body></html>";
  } catch (err) {
    pug.rethrow(err, pug_debug_filename, pug_debug_line);
  }
  return pug_html;
}
```

---

**THE END**






