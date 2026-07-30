# File Service — `Promact.FileService.Abstractions`

Generic file-storage abstraction over AWS S3 and Azure Blob Storage. Source:
`nuget-packages/file-service/` in
[Promact/reusable-components](https://github.com/Promact/reusable-components).

`IFileService<TFileModel>` is generic over the provider's own file-model
type, so the compiler enforces you're using the model matching your
registered provider — there's no runtime provider mismatch to worry about.

## Packages
| Provider | PackageId | File model |
|---|---|---|
| Core (always needed) | `Promact.FileService.Abstractions` | `IFileService<TFileModel>`, `FileModelBase`, `GetAllKeysRequest` |
| AWS S3 | `Promact.FileService.AWS` | `S3FileModel { BucketName }` |
| Azure Blob Storage | `Promact.FileService.Azure` | `AzureBlobFileModel { ContainerName }` |

## Public API
```csharp
namespace Promact.FileService.Abstractions;

public interface IFileService<TFileModel> where TFileModel : FileModelBase
{
    Task UploadFileAsync(TFileModel file, CancellationToken cancellationToken = default);
    Task<byte[]> GetFileAsBytesAsync(TFileModel file, CancellationToken cancellationToken = default);
    Task DeleteFileAsync(TFileModel file, CancellationToken cancellationToken = default);
    Task<string> GetSignedUrlAsync(TFileModel file, TimeSpan expiration, CancellationToken cancellationToken = default);
    Task<List<string>> GetKeysAsync(GetAllKeysRequest request, CancellationToken cancellationToken = default);
    Task<bool> CheckFileExistsAsync(TFileModel file, CancellationToken cancellationToken = default);
}

public abstract class FileModelBase { public required string FilePath; public required string KeyName; }
public class S3FileModel : FileModelBase { public required string BucketName; }
public class AzureBlobFileModel : FileModelBase { public required string ContainerName; }

public class GetAllKeysRequest { public required string BucketOrContainer; public int PageNumber; public int PageSize; }
```

## DI wiring — pick one provider

**AWS S3**
```csharp
services.AddAWSFileService(options =>
{
    options.AccessKey = configuration["AWS:AccessKeyId"];
    options.SecretKey = configuration["AWS:SecretAccessKey"];
    options.Region = configuration["AWS:Region"];
});
```
Config: `AWS:AccessKeyId`, `AWS:SecretAccessKey`, `AWS:Region` (required).
Registers `IFileService<S3FileModel>`.

**Azure Blob Storage**
```csharp
services.AddAzureFileService(options => options.ConnectionString = configuration["Azure:ConnectionString"]);
```
Config: `Azure:ConnectionString`. Registers `IFileService<AzureBlobFileModel>`.

## Usage
```csharp
using Promact.FileService.Abstractions;
using Promact.FileService.AWS; // or Promact.FileService.Azure

public class MyClass(IFileService<S3FileModel> fileService)
{
    public async Task DoWork()
    {
        var model = new S3FileModel { BucketName = "my_bucket", KeyName = "FileName", FilePath = "path/to/file.txt" };

        await fileService.UploadFileAsync(model);
        var exists = await fileService.CheckFileExistsAsync(model);
        var bytes = await fileService.GetFileAsBytesAsync(model);
        var url = await fileService.GetSignedUrlAsync(model, TimeSpan.FromHours(1));
        var keys = await fileService.GetKeysAsync(new GetAllKeysRequest { BucketOrContainer = "my_bucket", PageNumber = 1, PageSize = 50 });
        await fileService.DeleteFileAsync(model);
    }
}
```

## Gotchas
- **Azure signed URLs require anonymous blob read access enabled** on the
  storage account and container (Azure Portal → Storage Account →
  Configuration → "Allow Blob anonymous access" → Enabled, then on the
  container → Change Access Level → "Blob (Read Only)"). Without this,
  `GetSignedUrlAsync` may generate a SAS URL that still isn't reachable.
- `DeleteFileAsync` and `GetSignedUrlAsync` on **both** providers call
  `CheckFileExistsAsync` internally first and **throw
  `InvalidOperationException`** if the file/blob is missing — they are not
  no-ops on a missing file.
- `GetKeysAsync` requires `PageNumber` and `PageSize` both `> 0` (throws
  `ArgumentException` otherwise) and paginates **client-side from the start
  on every call** — watch cost/latency on large buckets/containers at high
  page numbers.
- Both providers wrap unexpected provider errors into a generic `Exception`
  — inspect `.InnerException` for the real underlying exception type.
- Every method takes an optional trailing `CancellationToken`.
