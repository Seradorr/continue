---
name: CSharp .NET Rules
description: |
  C#, .NET, WPF, WinForms, ASP.NET Core ve EF Core icin modern muhendislik kurallari.
globs:
  - "**/*.cs"
  - "**/*.xaml"
  - "**/*.csproj"
  - "**/*.sln"
  - "**/ViewModels/**/*"
  - "**/Views/**/*"
  - "**/Models/**/*"
  - "**/Services/**/*"
  - "**/Controllers/**/*"
  - "**/Migrations/**/*"
alwaysApply: false
---

# CSHARP .NET RULES

- Proje zaten kullaniyorsa mevcut solution, namespace ve dependency injection duzenine sadik kal.
- `async void` yalnizca UI event handler icin kullan; diger async akislar `Task` veya gerekliyse `ValueTask` donmeli.
- `CancellationToken` zincirini bozma; uygun yerde parametreyi ilet.
- Nullable reference type kurallarina uy; gereksiz `!` ile problemi gizleme.
- Public API veya service boundary degisiyorsa nullable, exception contract ve serialization etkisini birlikte dusun.
- Logging mevcut abstraction uzerinden yapilmali; stringly-typed ve baglamsiz logtan kacin.
- EF Core sorgularinda gereksiz eager load, N+1 veya tracking maliyetini fark etmeden arttirma.
- EF Core'da sorgu sekillendirmesi, transaction siniri ve `DbContext` omru birlikte dusunulmeli.
- WPF/MVVM tarafinda ViewModel, command ve binding sinirlarini karistirma.
- UI tarafinda thread affinity, async geri donus ve state guncellemelerini karistirma.
- `csproj`, startup, DI registration ve appsettings degisikliklerini sistem etkisi olan degisiklik olarak ele al.
- DI registration, options binding ve startup pipeline degisikliklerinde uygulama baslangic davranisini kontrol et.
- Verification onerisi olarak en az `dotnet build`, gerekiyorsa `dotnet test` veya ilgili uygulama akisini belirt.
