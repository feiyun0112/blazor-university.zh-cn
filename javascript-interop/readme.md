> 原文链接：https://blazor-university.com/javascript-interop/

# JavaScript 互操作
目前，WebAssembly 不支持许多功能，因此 Blazor 不提供对它们的直接访问。 这些通常是浏览器 API 功能，例如：

- [媒体捕捉](https://developer.mozilla.org/en-US/docs/Web/API/Media_Streams_API)
- [弹出窗口](https://www.w3schools.com/js/js_popup.asp)
- [Web GL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)
- [Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)

要访问这些浏览器功能，我们需要使用 JavaScript 作为 Blazor 和浏览器之间的中介； 这就是下一节所涵盖的内容。

## JavaScript 互操作警告
使用 JSInterop 时有一些注意事项。 这些将被添加到以下列表中，因为它们将在以后的部分中进行演示。

- [不要在服务器预渲染阶段调用 JSInterop](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-javascript-from-dotnet/updating-the-document-title#caveat)
- [不要过早使用 ElementReference 对象](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-javascript-from-dotnet/passing-html-element-references#caveat)
- [通过释放资源来避免内存泄漏](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-dotnet-from-javascript/lifetimes-and-memory-leaks/)
- [避免在已释放的 .NET 引用上调用方法](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-dotnet-from-javascript/lifetimes-and-memory-leaks/#caveat)
- [在 Blazor 初始化之前不要调用 .NET 方法](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-javascript-from-dotnet/passing-html-element-references/)

**[下一篇 - JavaScript 启动过程](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/javascript-boot-process/)**