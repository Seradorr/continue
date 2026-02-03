---
name: CSharp .NET Rules
description: |
  C#, .NET, WPF ve WinForms icin kod stili,
  async/await, MVVM ve anti-pattern kurallari.
globs:
  - "**/*.cs"
  - "**/*.xaml"
  - "**/*.csproj"
  - "**/*.sln"
  - "**/ViewModels/**/*"
  - "**/Views/**/*"
  - "**/Models/**/*"
  - "**/Services/**/*"
alwaysApply: false
---

# C# .NET KURALLARI

Bu kurallar C#, .NET, WPF ve WinForms gelistirme icin gecerlidir.

---

## 1. KOD STILI

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Class, Struct | PascalCase | UserService, OrderItem |
| Interface | I prefix | IUserRepository, ICommand |
| Method, Property | PascalCase | GetUserById, IsEnabled |
| Private field | _camelCase | _userService, _isReady |
| Local, Parameter | camelCase | userId, itemCount |
| Async method | Async suffix | GetDataAsync, SaveAsync |

### Dosya Yapisi

```csharp
using System;
using Microsoft.Extensions.Logging;
using MyProject.Models;

namespace MyProject.Services;

public class UserService : IUserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
    
    public async Task<User?> GetByIdAsync(int id, CancellationToken ct = default)
    {
        // Implementation
    }
}
```

---

## 2. ASYNC/AWAIT

### ConfigureAwait

```csharp
// Library: ConfigureAwait(false)
public async Task<Data> GetDataAsync()
{
    return await _client.GetAsync(url).ConfigureAwait(false);
}

// UI: ConfigureAwait yok (UI context gerekli)
private async void Button_Click(object sender, EventArgs e)
{
    var data = await GetDataAsync();
    TextBlock.Text = data.ToString();
}
```

### CancellationToken

```csharp
public async Task<Result> ProcessAsync(CancellationToken ct = default)
{
    ct.ThrowIfCancellationRequested();
    return await _service.DoWorkAsync(ct);
}
```

---

## 3. WPF / MVVM

### ViewModel Base

```csharp
public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler? PropertyChanged;
    
    protected bool SetProperty<T>(ref T field, T value, 
        [CallerMemberName] string? name = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value))
            return false;
        field = value;
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
        return true;
    }
}
```

### Property

```csharp
private string _userName = string.Empty;
public string UserName
{
    get => _userName;
    set => SetProperty(ref _userName, value);
}
```

---

## 4. ANTI-PATTERNS

### async void

```csharp
// YANLIS
public async void ProcessAsync() { }

// DOGRU
public async Task ProcessAsync() { }

// TEK ISTISNA: Event handler
private async void Button_Click(object sender, EventArgs e)
{
    try { await ProcessAsync(); }
    catch (Exception ex) { /* Handle */ }
}
```

### .Result ve .Wait()

```csharp
// YANLIS: Deadlock riski
var result = GetDataAsync().Result;
GetDataAsync().Wait();

// DOGRU
var result = await GetDataAsync();
```

### UI Thread Blocking

```csharp
// YANLIS
private void Button_Click(object sender, EventArgs e)
{
    Thread.Sleep(5000);  // UI donuyor!
}

// DOGRU
private async void Button_Click(object sender, EventArgs e)
{
    await Task.Delay(5000);
}
```

### IDisposable

```csharp
// YANLIS
var client = new HttpClient();
var result = client.GetAsync(url);
// Dispose yok!

// DOGRU
using var client = new HttpClient();
var result = await client.GetAsync(url);
```

### Bos Exception

```csharp
// YANLIS
try { DoSomething(); }
catch { }

// DOGRU
try { DoSomething(); }
catch (SpecificException ex)
{
    _logger.LogError(ex, "Failed");
    throw;
}
```

---

## 5. NULL HANDLING

```csharp
// Null check
if (value is not null) { }

// Null-conditional
var length = value?.Length ?? 0;

// Guard clause
ArgumentNullException.ThrowIfNull(user);
ArgumentException.ThrowIfNullOrWhiteSpace(name);
```

---

## 6. DEPENDENCY INJECTION

```csharp
public class OrderService : IOrderService
{
    private readonly IOrderRepository _repo;
    private readonly ILogger<OrderService> _logger;
    
    public OrderService(IOrderRepository repo, ILogger<OrderService> logger)
    {
        _repo = repo ?? throw new ArgumentNullException(nameof(repo));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}

// Registration
services.AddSingleton<IConfigService, ConfigService>();
services.AddScoped<IOrderService, OrderService>();
services.AddTransient<IEmailService, EmailService>();
```

---

## 7. MODERN C#

### Pattern Matching

```csharp
if (obj is string text) { }

var result = status switch
{
    Status.Active => "Running",
    Status.Stopped => "Finished",
    _ => "Unknown"
};
```

### Records

```csharp
public record User(int Id, string Name, string Email);
var updated = user with { Email = "new@email.com" };
```
