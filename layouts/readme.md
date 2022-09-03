> 原文链接：https://blazor-university.com/layouts/

# 布局
Blazor 布局类似于 ASP Webforms 母版页的概念，与 ASP MVC 中的 Razor 布局相同。

几乎网络上的每个网站都有一个模板用于整个网站（页面顶部的品牌，底部的版权）或网站的特定子部分（例如站点管理页面上的特定菜单结构）。

这是通过创建一个视图来实现的，该视图充当当前页面内容的 HTML 包装器，模板包含一个占位符，指示包装页面的内容应该出现在哪里。

```
<h1>This is the start of my reusable layout</h1>
<div class="Content">
  -- Some kind of indicator to specify the page's content will go here --
</div>
<footer>
  This is the end of the layout
</footer>
```

然后，各个页面可以选择指定一个布局，希望将其内容包含在其中。

```
-- Some way of indicating which template to wrap this page's content in --
<h1>This is the content of your embedded page</h1>
```

生成的 HTML 看起来像这样

```
<h1>This is the start of my reusable layout</h1>
<div class="Content">
  <h1>This is the content of your embedded page</h1>
</div>
<footer>
  This is the end of the layout
</footer>
```

**[下一篇 - 创建 Blazor 布局](https://feiyun0112.github.io/blazor-university.zh-cn/layouts/creating-a-blazor-layout)**