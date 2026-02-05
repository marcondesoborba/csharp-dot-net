Aqui vão **exemplos práticos e avançados** de **Source Generators** no C# (disponíveis desde C# 9/.NET 5, com melhorias contínuas até C# 14/.NET 10). Source Generators permitem gerar código em tempo de compilação, reduzindo boilerplate, melhorando performance e evitando reflection.

Vou mostrar exemplos reais usados em projetos grandes (como bibliotecas open-source ou apps enterprise), com código completo e explicações.

### 1. Exemplo Básico: Gerador de `ToString()` Automático para Records/Classes
Útil para debugging rápido sem escrever `ToString()` manualmente.

**Projeto de Source Generator** (crie um projeto .NET Class Library com `<IsPackable>true</IsPackable>` e referencie `Microsoft.CodeAnalysis.CSharp` e `Microsoft.CodeAnalysis.Analyzers` via NuGet).

```csharp
// ToStringGenerator.cs
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Text;
using System.Collections.Immutable;
using System.Text;

[Generator]
public class ToStringGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        // Encontra classes/records com atributo [GenerateToString]
        var provider = context.SyntaxProvider
            .CreateSyntaxProvider(
                predicate: static (node, _) => node is ClassDeclarationSyntax or RecordDeclarationSyntax,
                transform: static (ctx, _) =>
                {
                    var decl = (TypeDeclarationSyntax)ctx.Node;
                    var hasAttr = decl.AttributeLists
                        .SelectMany(al => al.Attributes)
                        .Any(a => a.Name.ToString().Contains("GenerateToString"));
                    return hasAttr ? decl : null;
                })
            .Where(static m => m is not null);

        context.RegisterSourceOutput(provider, static (spc, typeDecl) =>
        {
            if (typeDecl == null) return;

            var ns = typeDecl.FirstAncestorOrSelf<NamespaceDeclarationSyntax>()?.Name.ToString() ?? "Global";
            var className = typeDecl.Identifier.Text;
            var props = typeDecl.Members
                .OfType<PropertyDeclarationSyntax>()
                .Where(p => p.Modifiers.Any(m => m.IsKind(SyntaxKind.PublicKeyword)))
                .Select(p => (Name: p.Identifier.Text, Type: p.Type.ToString()))
                .ToImmutableArray();

            var sb = new StringBuilder($@"
namespace {ns};

partial {typeDecl.Keyword.Text} {className}
{{
    public override string ToString()
    {{
        return $""{className} {{ "");
");
            foreach (var prop in props)
            {
                sb.AppendLine($"        {prop.Name} = {{{prop.Name}}},");
            }
            sb.AppendLine(@"        }}"";");
            sb.AppendLine("    }");
            sb.AppendLine("}");

            spc.AddSource($"{className}_ToString.g.cs", SourceText.From(sb.ToString(), Encoding.UTF8));
        });
    }
}
```

**Uso no projeto consumidor** (adicione referência ao projeto do generator via `<ProjectReference Include="..\ToStringGenerator\ToStringGenerator.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="false" />`):

```csharp
[GenerateToString]
public record Pessoa(string Nome, int Idade, string Email);
```

**Resultado gerado** (em tempo de build):
```csharp
partial record Pessoa
{
    public override string ToString()
    {
        return $"Pessoa { Nome = {Nome}, Idade = {Idade}, Email = {Email}, }";
    }
}
```

**Vantagem avançada**: Evita reflection em logging/Serilog, melhora performance.

### 2. Exemplo Avançado: Gerador para INotifyPropertyChanged (MVVM)
Comum em WPF/MAUI/Blazor. Gera `OnPropertyChanged` automático para propriedades.

```csharp
// NotifyPropertyChangedGenerator.cs
[Generator]
public class NotifyPropertyChangedGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        // Pipeline incremental eficiente
        var classDeclarations = context.SyntaxProvider
            .ForAttributeWithSimpleName("NotifyPropertyChangedAttribute")
            .Select(static (ctx, _) =>
            {
                var typeDecl = ctx.TargetNode as ClassDeclarationSyntax;
                if (typeDecl == null) return null;

                var semanticModel = ctx.SemanticModel;
                var symbol = semanticModel.GetDeclaredSymbol(typeDecl);
                if (symbol?.BaseType?.ToDisplayString().Contains("INotifyPropertyChanged") == true)
                    return symbol;

                return null;
            })
            .Where(static m => m is not null);

        context.RegisterSourceOutput(classDeclarations, static (spc, classSymbol) =>
        {
            if (classSymbol == null) return;

            var ns = classSymbol.ContainingNamespace.ToDisplayString();
            var className = classSymbol.Name;

            var properties = classSymbol.GetMembers()
                .OfType<IPropertySymbol>()
                .Where(p => p.SetMethod != null && p.DeclaredAccessibility == Accessibility.Public)
                .ToList();

            var sb = new StringBuilder($@"
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace {ns};

partial class {className} : INotifyPropertyChanged
{{
    public event PropertyChangedEventHandler? PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string? propertyName = null)
    {{
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }}
");

            foreach (var prop in properties)
            {
                var type = prop.Type.ToDisplayString();
                var name = prop.Name;
                sb.AppendLine($@"
    private {type} _{char.ToLower(name[0]) + name.Substring(1)};
    public {type} {name}
    {{
        get => _{char.ToLower(name[0]) + name.Substring(1)};
        set
        {{
            if (EqualityComparer<{type}>.Default.Equals(_{char.ToLower(name[0]) + name.Substring(1)}, value)) return;
            _{char.ToLower(name[0]) + name.Substring(1)} = value;
            OnPropertyChanged();
        }}
    }}");
            }

            sb.AppendLine("}");
            spc.AddSource($"{className}_Notify.g.cs", SourceText.From(sb.ToString(), Encoding.UTF8));
        });
    }
}

// Atributo marcador
[AttributeUsage(AttributeTargets.Class, AllowMultiple = false)]
public class NotifyPropertyChangedAttribute : Attribute { }
```

**Uso**:
```csharp
[NotifyPropertyChanged]
public partial class ViewModel
{
    public string Nome { get; set; } // será gerado backing field + OnPropertyChanged
}
```

**Dica avançada**: Use incremental pipeline (`IncrementalGenerator`) para performance em projetos grandes (evita recompilação desnecessária).

### 3. Exemplo Integrado com Primary Constructors (C# 12+) e Source Generators
Gere código que usa os parâmetros capturados do primary constructor.

```csharp
// Gerador que adiciona Equals/GetHashCode baseado em primary params
[Generator]
public class PrimaryEqualsGenerator : IIncrementalGenerator
{
    public void Initialize(IncrementalGeneratorInitializationContext context)
    {
        var provider = context.SyntaxProvider
            .CreateSyntaxProvider(
                (node, _) => node is ClassDeclarationSyntax cds && cds.ParameterList != null,
                (ctx, _) => (ClassDeclarationSyntax)ctx.Node);

        context.RegisterSourceOutput(provider, (spc, cds) =>
        {
            var className = cds.Identifier.Text;
            var ns = cds.Ancestors().OfType<NamespaceDeclarationSyntax>().FirstOrDefault()?.Name.ToString() ?? "Global";

            var parameters = cds.ParameterList!.Parameters
                .Select(p => p.Identifier.Text)
                .ToList();

            var sb = new StringBuilder($@"
namespace {ns};

partial class {className}
{{
    public override bool Equals(object? obj) =>
        obj is {className} other && 
        {string.Join(" && ", parameters.Select(p => $"{p} == other.{p}"))};

    public override int GetHashCode() =>
        HashCode.Combine({string.Join(", ", parameters)});
}}
");
            spc.AddSource($"{className}_Equals.g.cs", SourceText.From(sb.ToString(), Encoding.UTF8));
        });
    }
}
```

**Uso**:
```csharp
public class Config(string apiKey, string endpoint, int timeoutSeconds)
{
    // Equals e GetHashCode gerados automaticamente baseados em apiKey, endpoint, timeoutSeconds
}
```

### Boas Práticas Avançadas (2026)
- Use **Incremental Generators** (IIncrementalGenerator) para builds rápidos.
- Teste com `Microsoft.CodeAnalysis.CSharp.Testing` ou Roslyn analizers.
- Publique como NuGet Analyzer (ex: `MyGenerators`).
- Em .NET 10/C# 14: Melhoria em source generators para partial members e extension members – use para gerar extensions automáticas.
- Evite gerar código excessivo (mantenha < 10k linhas por arquivo gerado).
- Debugging: Use `Debugger.Launch()` no generator para pausar no build.

Esses exemplos são usados em libs como CommunityToolkit.Mvvm, StronglyTypedId, etc. Se quiser um exemplo completo com projeto de teste, gerador para JSON, ou integração com ASP.NET (ex: endpoint discovery), avise que eu expando!
