# Atualização Framework

Neste guia, vamos abordar o processo de atualização de uma aplicação do .NET 3
para o .NET 6. O processo de atualização é simples, mas requer atenção para que 
não tenha nenhuma quebra de compatibilidade.

## Pré-requisitos
Antes de iniciar o processo de atualização, verifique se você tem o seguinte 
instalado em sua máquina:

- [.NET 6 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)
- [Visual Studio 2022](https://visualstudio.microsoft.com/pt-br/downloads/)

Para este guia, vamos utilizar o projeto [UpgradableAPI](https://github.com/Dalmazox/upgradable). As alterações aqui descritas
é como o projeto deverá ficar após atualizarmos.

## Passo 1: Atualizar o arquivo de projeto
O primeiro passo é atualizar o arquivo de projeto para a versão mais recente.

Para cada arquivo de projeto (extensão .csproj), temos de atualizar o `TargetFramework` para a versão mais recente. No nosso caso, vamos atualizar para `net6.0`.
```xml
<PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
</PropertyGroup>
```

Liste todos os pacotes desatualizados. Para isso, execute os comandos abaixo no terminal:
```bash
dotnet restore -f --no-cache
dotnet list package --outdated
```

Para atualizar os pacotes, execute o comando abaixo para cada pacote retornado no comando anterior:
```bash
dotnet add <nome-do-projeto> package <nome-do-pacote> --version <versão>
```

## Passo 2: Alterações em código

Aplique a alteração de top-level do arquivo `Program.cs`.

O porquê de fazermos essa alteração pode ser consultada [aqui](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/program-structure/top-level-statements).

```csharp
using System.Reflection;
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Serilog;
using Serilog.Events;
using Upgradable;

var builder = WebApplication.CreateBuilder(args);

var assembly = Assembly.Load("Upgradable.Application");

builder.Services.AddSwaggerGen();
builder.Services.AddMediatR(c => c.RegisterServicesFromAssembly(assembly));
builder.Services.AddScoped<IAccountService, AccountService>();
builder.Services.AddControllers();

Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Information()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
    .MinimumLevel.Override("System", LogEventLevel.Warning)
    .Enrich.FromLogContext()
    .WriteTo.Async(conf => conf.Console(
        outputTemplate:
        "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj} <s:{SourceContext}>{NewLine}{Exception}"))
    .CreateLogger();

builder.Host.UseSerilog();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(o => o.SwaggerEndpoint("/swagger/v1/swagger.json", "Upgradable"));
}

app.UseRouting();
app.UseEndpoints(e => e.MapControllers());
app.Run();
```
Mudar as classes imutáveis para o tipo [record](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record):

Ex.: `Accounts.cs`
```csharp
using System;

namespace Upgradable
{
    public record Account
    {
        public Guid AccountId { get; init; }
        public decimal Balance { get; init; }
    }
}
```
Incluir os file scoped [namespaces](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-10.0/file-scoped-namespaces):
```csharp
using System;

namespace Upgradable;

public class Account
{
    public Guid AccountId { get; set; }
    public decimal Balance { get; set; }
}
```
Outras alterações úteis vindas com o C# 10, podem ser acessadas [aqui](https://learn.microsoft.com/pt-br/dotnet/csharp/whats-new/csharp-10).

## Passo 3: Compilar e testar a aplicação
Após atualizar as dependências e o código, é hora de compilar e testar sua aplicação para garantir que tudo esteja funcionando corretamente. 
No Visual Studio 2022, compile seu projeto usando a opção "Build" no menu. Execute os testes unitários e de 
integração para garantir que tudo esteja funcionando corretamente.

# Conclusão
Neste artigo, abordamos o processo de atualização de uma aplicação do .NET 3 para o  .NET 6.
Certifique-se de seguir os passos mencionados e realizar testes adequados antes de implantar a versão 
atualizada em produção. Lembre-se de que cada aplicação é única e pode exigir alterações específicas. 
Consulte a documentação oficial do .NET 6 e as diretrizes específicas do seu projeto para obter orientações mais detalhadas.

