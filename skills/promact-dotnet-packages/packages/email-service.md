# Email Service — `Promact.EmailService`

Provider-agnostic email sending. One `IEmailService` interface, four
independent provider implementations. Source:
`nuget-packages/email-service/` in
[Promact/reusable-components](https://github.com/Promact/reusable-components).

## Packages
| Provider | PackageId | Notes |
|---|---|---|
| Core (always needed) | `Promact.EmailService` | Abstractions: `IEmailService`, `Email`, `EmailAddress`, `AttachmentData`, `TemplatedEmailRequest` |
| SMTP | `Promact.EmailService.SMTPS` | **Note the `S`** — nuget.org's `.SMTP` id was unavailable, so this one is `.SMTPS`. C# namespace is still `Promact.EmailService.SMTP`. |
| AWS SES | `Promact.EmailService.SES` | |
| SendGrid | `Promact.EmailService.SendGrid` | |
| Azure Communication Services | `Promact.EmailService.Azure` | |

## Public API
```csharp
namespace Promact.EmailService;

public interface IEmailService
{
    Task SendEmailAsync(Email email, CancellationToken cancellationToken = default);
    Task SendTemplatedEmailAsync(TemplatedEmailRequest templatedEmailRequest, CancellationToken cancellationToken = default);
}
```

## DI wiring — pick one provider

**SMTP**
```csharp
services.AddSMTPEmailService(options =>
{
    options.Host = configuration["SMTP:Host"];
    options.Port = int.Parse(configuration["SMTP:Port"]);
    options.UserName = configuration["SMTP:UserName"];
    options.Password = configuration["SMTP:Password"];
});
```
Config: `SMTP:Host`, `SMTP:Port`, `SMTP:UserName`, `SMTP:Password`.

**AWS SES**
```csharp
services.AddSESEmailService(options =>
{
    options.AccessKeyId = configuration["AWS:AccessKeyId"];
    options.SecretAccessKey = configuration["AWS:SecretAccessKey"];
    options.Region = configuration["AWS:Region"];
});
```
Config: `AWS:AccessKeyId`, `AWS:SecretAccessKey`, `AWS:Region` (required —
throws if empty). Leave key/secret blank to fall back to the default AWS
credential chain (IAM role, env vars, etc.).

**SendGrid**
```csharp
services.AddSendGridEmailService(options => options.APIKey = configuration["SendGrid:APIKey"]);
```
Config: `SendGrid:APIKey`.

**Azure Communication Services**
```csharp
services.AddAzureEmailService(options => options.ConnectionString = configuration["Azure:ConnectionString"]);
```
Config: `Azure:ConnectionString` (Azure Portal → Communication Service → Keys).

## Usage
```csharp
using Promact.EmailService;

public class MyClass(IEmailService emailService)
{
    public async Task SendAsync()
    {
        var email = new Email(
            to: [new EmailAddress("xyz@domain.com", "Receiver")],
            from: new EmailAddress("abc@pqr.com", "Sender"),
            subject: "Test subject", isBodyHtml: true, body: "Test body");
        email.Attachments!.Add(new AttachmentData(File.ReadAllBytes("path/to/file.txt"), "FileName", "text/plain"));
        await emailService.SendEmailAsync(email);
    }
}
```

Templated send (SES/SendGrid only — see gotchas):
```csharp
var request = new TemplatedEmailRequest(
    to: [new EmailAddress("xyz@domain.com", "Receiver")],
    from: new EmailAddress("abc@pqr.com", "Sender"),
    templateNameOrId: "welcome-email", // SES: template name; SendGrid: dynamic template ID
    templateData: new { Name = "Xyz", ActivationLink = "https://..." });
await emailService.SendTemplatedEmailAsync(request);
```

## Gotchas
- **SMTP and Azure do not support `SendTemplatedEmailAsync`** — both throw
  `NotImplementedException`. Only SES and SendGrid support templated sends.
- The sender email/domain must be **pre-verified with the provider** before
  sends will succeed (SES/SendGrid/SMTP: verify sender email; Azure: verify
  the Email Domain in Azure Communication Services).
- SES templated email renders the template locally with Handlebars.Net after
  fetching it via `GetTemplateAsync` — `TemplateData` should bind cleanly to
  the template's Handlebars placeholders.
