> 原文链接：https://blazor-university.com/javascript-interop/calling-dotnet-from-javascript/type-safety/

# 类型安全
在[从 JavaScript 调用 .NET](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-dotnet-from-javascript/) 部分中，您可能已经注意到我们的 JavaScript 的第 6 行在将随机生成的数字传递给 .NET 之前调用了 `toString()`

```
var BlazorUniversity = BlazorUniversity || {};
BlazorUniversity.startRandomGenerator = function(dotNetObject) {
  setInterval(function () {
    let text = Math.random() * 1000;
    console.log("JS: Generated " + text);
    dotNetObject.invokeMethodAsync('AddText', text.toString());
  }, 1000);
};
```

尽管对象类型在 JavaScript 中是可以互换的，但是当它们被传递给我们的 .NET `Invokable` 方法时，它们就不是那么互换了。调用 .NET 时，请确保为要传递的变量选择正确的 .NET 类型。

JavaScript 类型 |  .NET 类型
--- | ---
boolean	 | System.Boolean
string | 	System.String
number | 	System.Float / System.Decimal
Date | 	System.DateTime 或 System.String

## 枚举
当 `JSInvokable` .NET 方法的参数是枚举时，JavaScript 应该传递枚举的数值。下面的示例将使用值 `TestEnum.SecondValue` 调用我们的 .NET 方法。

```
public enum TestEnum
{
  FirstValue = 100,
  SecondValue = 200
};

[JSInvokable("OurInvokableDotNetMethod")]
public void OurInvokableDotNetMethod(TestEnum enumValue)
{
}
```

但是，如果我们用 `[System.Text.Json.Serialization.JsonConverter]` 装饰我们的枚举，我们可以让我们的 JavaScript 改为传递字符串值。

```
[System.Text.Json.Serialization.JsonConverter(typeof(System.Text.Json.Serialization.JsonStringEnumConverter))]
public enum TestEnum
{
  FirstValue = 100,
  SecondValue = 200
};
```

现在调用 JavaScript 可以传递枚举值的名称或其数值。以下两个调用是等效的。

```
dotNetObject.invokeMethodAsync('OurInvokableDotNetMethod', 'FirstValue');
dotNetObject.invokeMethodAsync('OurInvokableDotNetMethod', 200);
```

**[下一篇 - 调用静态 .NET 方法](https://feiyun0112.github.io/blazor-university.zh-cn/javascript-interop/calling-dotnet-from-javascript/calling-static-dotnet-methods/)**