# Beej's Guide源Markdown

## Beej对Markdown的扩展

这些是通过`bin/preproc`脚本实现的。

* `[[pagebreak]]`

  在打印版本中插入分页符。呈现为LaTeX的`\newpage`。

* `[nh[word]]`

  不允许对此单词进行连字符化。转换为`\hyphenation{word}`。由于LaTeX的限制，下划线是被禁止的。

* `[ix[entry]]`

  构建一个LaTeX索引条目，将其原样替换为`\index{}`元素。**记得在这些条目中用`\_`转义你的下划线！** [LaTeX索引示例请点击这里](https://en.wikibooks.org/wiki/LaTeX/Indexing#Sophisticated_indexing)。

* `[ixtt[entry]]`

  对于单个、简单的索引条目，使用`\index{entry@\texttt{entry}}`在等宽文本中呈现。 更复杂的条目应通过将原始LaTeX传递给`[ix[]]`进行渲染。

* `[fl[linktext|url]]`

  脚注链接。将linktext超链接到url，并在脚注中显示URL。转化为`[linktext](url)^[url]`。

* `[flx[linktext|file]]`

  脚注链接到Beej的示例。自动在文件之前添加`https://beej.us/guide/bgnet/examples/`并在脚注中显示URL。

* `[flr[link|id]]`

   脚注链接到Beej的重定向页面。自动在链接id之前添加`https://beej.us/guide/url/`并在脚注中显示URL。

* `[flrfc[link|num]]`

   脚注链接到RFC。自动在RFC编号之前添加`https://tools.ietf.org/html/rfc`，并在脚注中显示URL。
   

## pandoc markdown问题

如果在同一段落中有多个内联脚注，但它们位于不同的markdown行上，则除了最后一行外的所有行都必须以尾随空格结尾。

文档页面子节为 h4 `####`，以防显示在目录中。 Pandoc 2.8 应该有一种方法可以在目录中隐藏 h3。

表格行需要放在一行中以正确换行（表格单元格中的换行保留）。

表格：表头的相对宽度会反映在最终输出中。

表格头模板：

```
| 宏名            | 描述                                                  |
|-----------------|-------------------------------------------------------|
```

`<a name>` 不起作用。请使用标题 `{#tags}`。

使用标准语言名称的围栏代码有时会导致 LaTeX 无法工作。

````
```c                   不要这样做

``` {.c}               这样做
``` {.c .numberLines}  或这样
````

索引使用 LaTeX 标记 `\index{foo}` 进行。它们不会出现在 HTML 输出中。

LaTeX 索引示例：

（使用 `\_` 转义下划线！）

```latex
\index{foo} 普通元素
\index{foo\_bar} 带下划线的元素
\index{foo()@\texttt{foo()}} 在索引中以等宽字体显示 foo()
\index{O\_NONBLOCK@\texttt{O\_NONBLOCK}} 带下划线的等宽字体
\index{foo!bar} foo 中的子索引 bar
\index{bind()@\texttt{bind()}!implicit} 带等宽字体的子索引
```

[更多 LaTeX 索引示例请参见此处](https://en.wikibooks.org/wiki/LaTeX/Indexing#Sophisticated_indexing)。

索引条目应在主要级别，而不是在标题、粗体或斜体文本中（否则它们会显示为单独的索引条目）。

在每个 manpage 前面放置 `\newpage` 以强制分页。

在编辑 `bgnet_amazon.md` 时，使用 `\newpage` 来强制将断页符移到下一页。

在顺序行中链接后的脚注末尾需要有一个空格才能正常工作...？

```markdown
[嘿](url)^[脚注]，   <-- 这行末尾需要有一个空格
[再次](url2)^[脚注2]。
```