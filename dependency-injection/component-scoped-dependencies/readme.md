> 原文链接：https://blazor-university.com/dependency-injection/component-scoped-dependencies/

# 组件范围依赖
到目前为止，我们已经了解了三种依赖注入作用域：Singleton、Scoped 和 Transient。 我们还尝试了解这些[不同的依赖注入作用域如何相互比较](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/dependency-lifetimes-and-scopes/comparing-dependency-scopes/)，以及 ASP.NET MVC 和 Blazor 之间的[作用域生命周期有何不同](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/dependency-lifetimes-and-scopes/scoped-dependencies/)。

在某些情况下，我们可能需要对注入依赖项的生命周期进行更多控制，并控制它们是跨组件共享还是仅供单个组件使用。 以下部分将涵盖一些场景以及如何实施它们； 其中一些已经内置到 Blazor 中，还有一些是根据我们在此过程中学到的内容定制的。


**[下一篇 - OwningComponentBase<T>](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/component-scoped-dependencies/owningcomponentbase-generic/)**