> 原文链接：https://blazor-university.com/dependency-injection/

# 依赖注入
## 依赖注入概述
依赖注入是确保类保持松散耦合并使单元测试更容易的最佳实践软件开发技术。

以使用第三方服务发送电子邮件的服务为例。传统上，任何需要使用此服务的类都可能创建一个实例。

```
public class NewsletterService
{
  private readonly IEmailService EmailService;

  public NewsletterService()
  {
    EmailService = new SendGridEmailService();
  }

  public void SignUp(string emailAddress)
  {
     EmailService.SendEmail("noreply@sender.com", emailAddress, "Subject", "Body");
  }
}
```

这种方法的问题在于它紧密耦合类 `NewsletterService` 和 `SendGridEmailService`。在对 `MyClass.SignUp` 进行单元测试时，被测试的方法实际上会尝试发送电子邮件。这不仅对您的收件箱不利，而且对成本不利（如果您的提供商对每封电子邮件收费），并且实际上我们只需要知道 `SignUp` 方法试图发送电子邮件以欢迎新用户使用我们的服务。

使用依赖注入而不是每个消费类都必须创建正确的 `IEmailService` 实现程序的实例，我们的消费类期望在创建时提供正确的实例。

```
public class NewsletterService
{
  private readonly IEmailService EmailService;

  public NewsletterService(IEmailService emailService)
  {
    EmailService = emailService;
  }

  public void SignUp(string emailAddress)
  {
     EmailService.SendEmail(...);
  }
}
```

当我们要求它为我们构建 `NewsletterService` 的实例时，依赖注入框架（例如在 ASP.NET MVC 应用程序和 Blazor 应用程序中默认使用的那个）将自动注入正确类的实例。

这不仅通过使 `NewsletterService` 不知道实现 `IEmailService` 的类来解耦我们的类，而且还使单元测试变得非常简单。例如，使用 Moq 框架。

```
[Fact]
public void WhenSigningUp_ThenSendsAnEmail()
{
  var mockEmailService = new Mock<IEmailService>();
  
  var subject = new NewsletterService(mockEmailService.Object);
  subject.SignUp("Bob@Monkhouse.com");

  mockEmailService
    .Verify(
      x => x.Send("noreply@sender.com", "Bob@Monkhouse.com", "Subject", "Body"),
      Times.Once);
}
```


**[下一篇 - 将依赖项注入 Blazor 组件](https://feiyun0112.github.io/blazor-university.zh-cn/dependency-injection/injecting-dependencies-into-blazor-components/)**