```c
// 用这个宏保存变量值
#define SAVE_VAR(x, val) do {         // 用这个宏保存[x]的值
    int temp = (val);                // 用这个宏保存[x]的值
    save_var(#x, temp);              // 用这个宏保存[x]的值
} while (0)                           // 用这个宏保存[x]的值
```

```simplified
man页小节使用h4 `####`来避免出现在目录中。Pandoc 2.8 应该有方法来阻止h3显示在目录中。

表格行需要放在一行上以便正确换行（表格单元格中保留换行符）。

表格：标题的相对宽度会反映在最终输出中。

表头模板：

```
| Macro           | Description                                            |
|-----------------|--------------------------------------------------------|
```

`<a name>`不起作用。使用标题 `{#标签}`。

用标准语言名称固定代码块有时会导致LaTeX出问题。

````
```c                   不要这样做

``` {.c}               这样做
``` {.c .numberLines}  或者这样
````

使用LaTeX的 `\index{foo}` 标记进行索引。它们不会显示在HTML输出中。

LaTeX索引示例：

(使用 `\_` 转义下划线!)

```latex
\index{foo} plain element
\index{foo\_bar} 带下划线的元素
\index{foo()@\texttt{foo()}} 在索引中以等宽字体显示foo()
\index{O\_NONBLOCK@\texttt{O\_NONBLOCK}} 带下划线的等宽字体
\index{foo!bar} foo中的子索引bar
\index{bind()@\texttt{bind()}!implicit} 带等宽字体的子索引
```

[更多LaTeX索引示例请点击这里](https://en.wikibooks.org/wiki/LaTeX/Indexing#Sophisticated_indexing)。

索引项应该位于主级别，而不是在标题或粗体或斜体文本中（否则它们会作为单独的条目显示在索引中）。

在每个man页之前加上 `\newpage` 来强制分页。

编辑 `bgnet_amazon.md` 时，使用 `\newpage` 来将孤行移到下一页。

链接后面的脚注应该在连续的行上，最后一行需要有一个空格才能正常工作...?

```markdown
[Hey](url)^[脚注],   <-- 这一行末尾需要一个空格
[Again](url2)^[脚注2].
```
```