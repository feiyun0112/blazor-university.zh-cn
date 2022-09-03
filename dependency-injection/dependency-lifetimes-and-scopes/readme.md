> 原文链接：https://blazor-university.com/dependency-injection/dependency-lifetimes-and-scopes/

# 依赖生命周期和范围
使用依赖注入时要问的两个重要问题是：这些依赖项的实例将存在多长时间，以及有多少其他对象可以访问同一实例？

答案取决于许多因素，主要是因为我们提供了不同的选择，以便我们可以自己做出决定。 第一个因素，可能也是最容易理解的因素，是注册依赖项的作用域。 `Microsoft.Extensions.DependencyInjection.ServiceLifetime` 定义的范围是 `Singleton`、`Scoped` 和 `Transient`。

**[下一篇 - Transient 依赖](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/dependency-lifetimes-and-scopes/transient-dependencies/)**