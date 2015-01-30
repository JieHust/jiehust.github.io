---
layout: post
category : notes
tagline: "Tips and Problems"
tags : [Jekyll, markdown, latex, highlighter]
use_toc : true
---

本文用于记录使用 Jekyll 以及 Github Pages 搭建个人博客时常用一些技巧以及常见的一些问题。关于[如何搭建](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)个人博客以及其[工作原理](http://jekyllbootstrap.com/lessons/jekyll-introduction.html)可以参考 [Jekyll Bootstrap](http://jekyllbootstrap.com)。对于完全不同前端技术的同学（像我一样的同学 - -||），可以先在 [codecademy](http://www.codecademy.com/) 学习关于 html 以及 css 的基本知识即可。

### 使用 Latex 

```html
<script type="text/javascript" src="path-to-mathjax/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
```

Mathjax 默认独占一行的公式为 `$$...$$` 和 `\[...\]`，行内公式为 `\(...\)`,而用习惯了 latex 中函内公式 `$...$` 可以对 Mathjax 进行[配置](http://docs.mathjax.org/en/latest/options/tex2jax.html#configure-tex2jax)，如下：

```html
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
}); </script>
```

**注意1:** 由于下划线 `_` 和 星号 `*` 在 markdown 中有特殊含义 （**成对**出现时，通常别解析为 `<em>` 或 `<strong>`）， markdown 对原文档进行解析以后生成 html，mathjax 再在对解析以后的 html 内容进行公式解析。当复杂公式中**出现多个特殊符号**，容易被 markdown 解析为 `<em>` (并不是所有情况都会别解析为` <em>`，而是于符号出现的个数与位置)，此时，需在前面加转移符号 `\`。

**注意2:** 不同的 markdown 处理器，对公式中出现的成对的特殊字符处理方式略有不同。 Jekyll 中支持四种 markdown 处理器。本人试过的 `kramdown` 与 `redcarpet`。

> **kramdown**:
>
>  - **行间公式:** 对 `_` 不用加转义符（若加了，则转移符将被输出，公式异常），而 `*` 需加转义符，
>  - **行内公式:** 必须加转移符号。
> 
> **redcarpet**: 
>  
>  - 所有特殊符号都加上 `\`，就不会出问题。不加得看个数与位置的情况。 

总之，使用与github上相同的环境调试并查看网页源码就很容易发现转义引起的相关问题，再做调整。如果中途更换 markdown 处理器，之前博文中的复杂公式可能会出现问题。对于公式编辑，可以借助[在线的 latex 编辑器](http://www.codecogs.com/latex/eqneditor.php)，免去本地搭建 latex 环境，且在线编辑器方便，只要有网即可工作，其功能对于博客写作已是绰绰有余。

### 代码高亮

如果在文档中使用 Liquid 语法支持的代码高亮，可以使用：

```
{% raw %}{% highlight python %}
def yourfunction():
    print "Hello World!"
{% endhighlight %}{% endraw %}
```
其效果为：

{% highlight python %}
def yourfunction():
    print "Hello World!"
{% endhighlight %}

若需要带行号：

```
{% raw %}{% highlight python linenos %}   
{% endhighlight %}{% endraw %}
```

效果为：

{% highlight python linenos %}
def yourfunction():
    print "Hello World!"
{% endhighlight %}


这种方式使用了非 markdown 的语法，对于 markdown 处理器是无法对其处理的，在 github page 上，其由 Jekyll 先使用 Liquid 解析以后，再经由 markdown 处理器。其缺点也就很明显，当你离线编辑或不使用 Jekyll 时，你的文档排版将会很丑。

另一种方法即为使用 `pygments` 来进行代码高亮， 首先选择支持围栏式（Fenced）的代码块的markdown 处理器 `redcarpet` 或 `RDiscount` 作为 Jekyll 中的处理器。其特征是，可利用开始和结束围栏行（fence lines）而不是缩进将代码围起来。围栏行包含三个或者更多波浪线`~`或反引号`` ` ``,开始行可以为代码添加语言描述。开始行和结束行所使用的符号必须相同数量相等。需要注意的是，开始行的前面必须有空行，否则将转换为行内代码。结束行之后可以没有空行。以 `redcarpet` 为例，对于`_config.yml`的配置如下：

```
highlighter: pygments
markdown: redcarpet
redcarpet:
  extensions: ["fenced_code_blocks"]
```

代码块即可使用(请忽略`\`):

````
 
\```LANGUAGE
\  write code here
\```
````

若使用 kramdown 解析代码段，其只能解析缩进的代码段，且解析的结果的源码为

```html
<pre><code>union data{
    int a;
}
</code></pre>
```

效果如下：

```
union data{
    int a;
}
```


当使用 `redcarpet` 与 `pygments` 进行上述配置后，其解析结果的源码为

```html
<div class="highlight"><pre><code class="language-C" data-lang="C"><span class="k">union</span> <span class="n">data</span><span class="p">{</span>
    <span class="kt">int</span> <span class="n">a</span><span class="p">;
<span class="p">}</span>
</code></pre></div>
```

此时代码中的文字都加入了相应的 `class="name"`,再使用相应的 css 来对不同类进行设置，以达到代码高亮的效果。利用生成pygmentize生成 css 文件：

```
pygmentize -S default -f html > pygments.css
```

将生成的 css 加入到相应的 post 模板内即可。 效果如下：

```C
union data{
    int a;
}
```

### 图片居中

markdown 对图片的支持过于简单，可以通过 html 方式来添加对图片控制的支持，如大小和位置。

**居中：**
在 css 中添加以下代码：

```css
.aligncenter {
  clear: both;
  display: block;
  margin:auto;
}
```

**添加图片：**

```html
<img class="aligncenter" src="path_to/image.jpg" alt="Drawing" style="width: 300px;" align="center"/>
```
