+++
date = '2025-07-22T21:20:00+08:00'
draft = false
title = 'Markdown and latex'
katex = true
+++

本文 markdown 相关的内容主要参考网站 [Markdown教程](https://markdown.com.cn/)，记下一些自己常用常忘的语法。至于 latex，GPT 是我的唯一指定 latex 导师。这个框架下 markdown 的渲染不等价与所有框架的 markdown 渲染，并且数学公式的渲染也与真实 latex 环境有些差异，但无关紧要。

题外话，这个框架下，如果你要写二级或者三级标题建议采取 ```2n``` 个 \#，如果只差一个 \#，感觉标题层次不太清晰。但是这样设置会影响目录的自动生成，如果想折腾的话建议自己写一个 scss 更改标题的大小。

## Latex

### 常用数学符号表

#### 分析代数

| 代码 | 效果 | 描述 |
| :---: | :---: | :---: |
| ```\|·\|``` | $ \|·\| $ | 范数，这里渲染不太对，请直接看源码 |
| ```\nabla``` | $ \nabla $ | \ |
| ```\partial``` | $ \partial $ | \ |
| ```\mathbb{R}``` | $ \mathbb{R} $ | 渲染不够华丽啊，而且很慢 |
| ```\to``` | $ \to $ | \ |
| ```\mapsto``` | $ \mapsto $ | \ |

#### 大小比较

| 代码 | 效果 | 描述 |
| :---: | :---: | :---: |
| ```\geq``` | $ \geq $ | \ |
| ```\leq``` | $ \leq $ | \ |
| ```\neq``` | $ \neq $ | \ |

#### 因果推理

| 代码 | 效果 | 描述 |
| :---: | :---: | :---: |
| ```\forall``` | $ \forall $ | \ |
| ```\exists``` | $ \exists $ | \ |
| ```\because``` | $ \because $ | \ |
| ```\therefore``` | $ \therefore $ | \ |
| ```\Leftarrow``` | $ \Leftarrow $ | \ |
| ```\Leftrightarrow``` | $ \Leftrightarrow $ | \ |
| ```\neg``` | $ \neg $ | \ |
| ```\in``` | $ \in $ | \ |

#### 希腊字符

| 代码 | 效果 | 描述 |
| :---: | :---: | :---: |
| ```\lambda``` | $ \lambda $ | \ |

#### 其他

## Markdown

### 空格

文字段落内空格 ```$~~~$```，这里是 $~~~$ 效果。

### 制表

```
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
```

| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |

一个简易的表格示意，第二行指定了对齐方式。

### Emoji

直接从 [Emojipedia](https://emojipedia.org/) 中复制粘贴就好了。下面粘贴一些我用过的表情。类我就不分了，应该用不了几种。

⭐🥰😅🙃😋😭