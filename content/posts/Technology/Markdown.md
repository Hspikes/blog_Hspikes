+++
date = '2025-07-22T21:20:00+08:00'
draft = false
title = 'Markdown and latex'
katex = true
+++

本文 markdown 相关的内容主要参考网站 [Markdown教程](https://markdown.com.cn/)，记下一些自己常用常忘的语法。至于 latex，GPT 是我的唯一指定 latex 导师。这个框架下 markdown 的渲染不等价与所有框架的 markdown 渲染，并且数学公式的渲染也与真实 latex 环境有些差异，但无关紧要。

题外话，这个框架下，如果你要写二级或者三级标题建议采取 ```2n``` 个 \#，如果只差一个 \#，感觉标题层次不太清晰。但是这样设置会影响目录的自动生成，如果想折腾的话建议自己写一个 scss 更改标题的大小。

## Latex

### 数学符号表

#### 分析代数

| 代码 | 效果 |
| :---: | :---: |
| ```\|·\|``` | $ \|·\| $ 范数，这里渲染不太对，请直接看源码 |
| ```\nabla``` | $ \nabla $ |
| ```\partial``` | $ \partial $ |
| ```\to``` | $ \to $ |
| ```\mapsto``` | $ \mapsto $ |
| ```\int``` | $ \int $ |
| ```\iint``` | $ \iint $ |
| ```\oint``` | $ \oint $ |
| ```\iint\limits_{D}``` | $ \iint\limits_{D} $ 在积分号正下方 |

#### 大小比较

| 代码 | 效果 |
| :---: | :---: |
| ```\geq``` | $ \geq $ |
| ```\leq``` | $ \leq $ |
| ```\neq``` | $ \neq $ |

#### 因果推理

| 代码 | 效果 | 
| :---: | :---: |
| ```\forall``` | $ \forall $ |  
| ```\exists``` | $ \exists $ |  
| ```\because``` | $ \because $ |  
| ```\therefore``` | $ \therefore $ |  
| ```\Leftarrow``` | $ \Leftarrow $ |  
| ```\Leftrightarrow``` | $ \Leftrightarrow $ |  
| ```\neg``` | $ \neg $ |  
| ```\in``` | $ \in $ |  

#### 希腊字符

| 代码 | 效果 |
| :---: | :---: |
| ```\lambda``` | $ \lambda $ |  
| ```\Lambda``` | $ \Lambda $ |  
| ```\alpha``` | $ \alpha $ |  
| ```\beta``` | $ \beta $ |  
| ```\gamma``` | $ \gamma $ |  
| ```\Gamma``` | $ \Gamma $ |  
| ```\delta``` | $ \delta $ |  
| ```\Delta``` | $ \Delta $ |  
| ```\epsilon``` | $ \epsilon $ |  
| ```\omega``` | $ \omega $ |  
| ```\phi``` | $ \phi $ |  
| ```\Phi``` | $ \Phi $ |  
| ```\mu``` | $ \mu $ |  
| ```\eta``` | $ \eta $ |  
| ```\theta``` | $ \theta $ |  
| ```\xi``` | $ \xi $ |  
| ```\Xi``` | $ \Xi $ |  

#### 上标修饰符

| 代码 | 效果 |
| :---: | :---: |
| ```\tilde(x)``` | $ \tilde{x} $ |
| ```\bar``` | $ \bar{x} $ |
| ```\hat``` | $ \hat{x} $ |
| ```\vec{\alpha}``` | $ \vec{\alpha} $ | 
| ```\triangleq``` | $ \triangleq $ 定义为 | 

也可以自定义的写法，可以更为灵活的决定符号

```a \stackrel{\triangle}{=} b```

```a \stackrel{def}{=} b```

```a \stackrel{\text{def}}{=} b```

效果如下：

$ a \stackrel{\triangle}{=} b $

$ a \stackrel{def}{=} b $

$ a \stackrel{\text{def}}{=} b $

#### 矩阵

```tex
\begin{bmatrix}
    \frac{\partial^2 f}{\partial x^2} & \frac{\partial^2 f}{\partial x \partial y} \\
    \frac{\partial^2 f}{\partial y \partial x} & \frac{\partial^2 f}{\partial y^2} 
\end{bmatrix}
```
<div>\[
\begin{bmatrix}
    \frac{\partial^2f}{\partial x^2} & \frac{\partial^2 f}{\partial x \partial y} \\
    \frac{\partial^2 f}{\partial y \partial x} & \frac{\partial^2 f}{\partial y^2} 
\end{bmatrix}
\]</div>


#### 方程组(公式块内换行)

latex 本身有非常多神秘的设计，比如你单输入一个 _ 在文本中是无法通过编译的，因为它觉得你输入 _ 是忘记打公式块了。包括 latex 的公式块是不支持块内用 ```\\``` 换行的，非常离谱。

要实现公式块内换行，只能用额外的块嵌套。

```tex
\[
\begin{aligned}
    &\min_x f_0(x) + \sum_{i=1}^{m} I_-(f_i(x)) \\
    & s.t. Ax = b
\end{aligned}
\]
```
<div>$$
\begin{aligned}
    &\min_x f_0(x) + \sum_{i=1}^{m} I_-(f_i(x)) \\
    & s.t. Ax = b
\end{aligned}
$$</div>

要实现大括号方程组也必须要依赖与这个方案。

```tex
\[
\left\{
\begin{aligned}
    & A\,x^*(t) + b = 0,\\
    & f_i(x^*(t)) < 0,\quad i=1,2,\dots,m,\\
    & t\nabla f_0(x^*(t)) 
    + \sum_{i=1}^{m}\frac{1}{-f_i(x^*(t))}\nabla f_i(x^*(t)) 
    + A^\top v = 0.
\end{aligned}
\right.
\]
```

<div>$$
\left\{
\begin{aligned}
    & A\,x^*(t) + b = 0,\\
    & f_i(x^*(t)) < 0,\quad i=1,2,\dots,m,\\
    & t\nabla f_0(x^*(t)) 
    + \sum_{i=1}^{m}\frac{1}{-f_i(x^*(t))}\nabla f_i(x^*(t)) 
    + A^\top v = 0.
\end{aligned}
\right.
$$</div>


注意代码中的 ```\right.``` 就代表右侧为空字符。

其实这个方案也可以用于实现矩阵，但矩阵有更方便的实现，没有必要。在写这个博客的时候我发现我本地的 markdown 也非常神秘，注意 ```\begin{aligned}... \end{aligned}``` 之间我有缩进，没有缩进无法渲染，也可能是我的渲染器的问题吧。

还有一个非常神秘的点是我本地渲染器渲染完有用 hugo 本地渲染了一下，发现只要有复杂的格式控制的公式块前必须加入 ```<div>``` 才能正常渲染，有没有缩进又不重要了，```\[\] or $$$$``` 也不重要，而且后面加不加 ```</div>```也不重要。而且我本地的渲染器无论是 typora 还是 vscode 的插件在加入 ```<div>``` 后都无法正确渲染了，太神秘了。

### 文本效果

数学符号部分由于 markdown 也是内嵌的 ketex，所以文中数学符号的渲染和 latex 中的渲染大致相同。

但是在文本效果部分显然 markdown 不会像 latex 一样渲染 ~~(而且 markdown 方便多了)~~，所以渲染效果就只能用语言来描述了。

| 代码 | 效果 |
| :---: | :---: |
| ```\textbf{}``` | 加粗 |
| ```\texttt{}``` | 等宽字体，虽然说是用在插入代码的，但实际效果不太理想，插入代码远不如 md 方便 |
| ```\mathbb{R}``` | $ \mathbb{R} $ 渲染不够华丽啊，而且很慢 |
| ```\mathcal{L}``` | $ \mathcal{L} $ |

### 枚举

最推荐使用 ```usepackage{enumitem}``` 宏包，自定义方式多样化。

基本格式为 ```\begin{enumerate}[label=...] \item... \end{enumerate}```，标签的书写就在```label```中完成。下面举一些常用标签的例子：

| 代码 | 效果 |
| :---: | :---: |
| ```\arabic*``` | 1, 2, 3... |
| ```\alph*``` | a, b, c... |
| ```\Alph*``` | A, B, C... |
| ```\roman*``` | i, ii, iii... |
| ```\Roman*``` | I, II, III... |
| ```\textbullet``` | · |
| ```--``` | - |
| ```步骤 \arabic*``` | 步骤 1, 步骤 2... |
| ```(\arabic*)``` | (1), (2), (3)... |

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

### 图片插入

编写旅游博客需要插入图片，经过了漫长的折腾终于折腾出了一个理想的效果，痛定思痛决定定下模版，以后直接采用。

如果图片需要旋转不要在文档中旋转。

```css
/* 单张图片，横竖不同情况下可以适当调节比例，好看为主 */
<div align=center>  
<img src="1.jpg" style="width: 75%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">  
</div>

/* 两图并排 */
<div style="display: flex; justify-content: center; gap: 0px 0px; flex-wrap: wrap; line-height: 0;">
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">  
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
</div>

/* 三图，行间距控制*/
<div style="display: flex; justify-content: center; gap: 0px 0px; flex-wrap: wrap; line-height: 0;">
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">  
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">

<img src="1.jpg" style="width: 60%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin-top: -10px;">
</div>

/* 四图，行间距控制*/
<div style="display: flex; justify-content: center; gap: 0px 0px; flex-wrap: wrap; line-height: 0;">
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">  
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">  
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin-top: -10px;">  
<img src="1.jpg" style="width: 48%; border-radius: 8px; border: 5px solid white; box-shadow: 0 4px 8px rgba(0,0,0,0.1); margin-top: -10px;">  
</div>
```