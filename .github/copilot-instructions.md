# GitHub Copilot Instructions for C# / .NET Repository

## Repository Overview

This repository contains an educational e-book guiding developers from .NET Framework 4.5 to .NET 10 and C# 14. The content is written in Brazilian Portuguese and focuses on modern .NET development practices.

## Language and Documentation

- **Primary Language**: Brazilian Portuguese (pt-BR)
- All documentation, comments, and text content should be in Portuguese
- Code examples should follow C# naming conventions (PascalCase for public members, even when text is in Portuguese)

## C# and .NET Standards

### Target Framework
- Use .NET 10 as the reference platform
- Target C# 14 language features when providing examples
- Prefer modern C# syntax and patterns

### Code Style

#### Naming Conventions
- Use PascalCase for classes, methods, properties, and public members
- Use camelCase for private fields and local variables
- Use descriptive names in Portuguese when appropriate for domain concepts
- Use English for technical/framework terms

#### Modern C# Features (Prioritize when applicable)
- **Primary Constructors**: Use for dependency injection and simple initialization
- **Collection Expressions**: Prefer `[1, 2, 3]` over `new[] { 1, 2, 3 }`
- **Pattern Matching**: Use switch expressions and pattern matching for cleaner code
- **Records**: Use for DTOs and immutable data structures
- **Nullable Reference Types**: Enable and use `?` annotations appropriately
- **File-scoped Namespaces**: Prefer `namespace MyApp;` over nested blocks
- **Global Usings**: Consolidate common usings in a single file
- **Top-level Statements**: Use for simple console applications
- **Init-only Properties**: Use `init` for immutable properties after construction
- **Required Members**: Use `required` keyword for mandatory properties

#### Example Code Style
```csharp
// Good: Modern C# 14 style
namespace MinhaApp;

public class UsuarioService(ILogger<UsuarioService> logger, IUsuarioRepository repository)
{
    public async Task<Usuario?> ObterPorIdAsync(int id)
    {
        logger.LogInformation("Buscando usuário {Id}", id);
        return await repository.ObterPorIdAsync(id);
    }
}

// Good: Records for DTOs
public record Usuario(string Nome, string Email, int Idade);

// Good: Pattern matching
public string ClassificarUsuario(Usuario usuario) => usuario switch
{
    { Idade: < 18 } => "Menor de idade",
    { Idade: >= 18 and < 60 } => "Adulto",
    { Idade: >= 60 } => "Idoso",
    _ => "Desconhecido"
};
```

### ASP.NET Core Preferences

- **Minimal APIs**: Prefer Minimal APIs over traditional Controllers for new REST endpoints
- **Dependency Injection**: Always use constructor injection (primary constructors when possible)
- **Async/Await**: All I/O operations should be async
- **Logging**: Use structured logging with `ILogger<T>`
- **Configuration**: Use `IOptions<T>` pattern for configuration

### Performance and Best Practices

- Prefer `System.Text.Json` over Newtonsoft.Json
- Use `Span<T>` and `Memory<T>` for performance-critical operations
- Avoid boxing/unboxing when possible
- Use `ValueTask<T>` for frequently synchronous async operations
- Enable Native AOT compilation considerations when applicable

### Testing Preferences

- Use xUnit for unit tests (if tests are added)
- Follow AAA pattern (Arrange, Act, Assert)
- Use meaningful test method names in Portuguese
- Mock dependencies using interfaces

### Documentation

- Use XML documentation comments for public APIs
- Include code examples in documentation when beneficial
- Keep documentation in Portuguese for this repository
- Reference official Microsoft documentation when appropriate

## Project-Specific Guidelines

### Educational Content
- When adding code examples, include comments explaining modern .NET features
- Compare old (.NET Framework 4.5) vs new (.NET 10) approaches when relevant
- Highlight performance improvements and modern best practices
- Include migration tips and common pitfalls

### Markdown Formatting
- Use proper headings hierarchy
- Include code blocks with language specification (```csharp)
- Use tables for comparisons when appropriate
- Keep line length reasonable for readability

### File Organization
- Main e-book content is in `EBOOK.md`
- README provides repository overview
- Keep repository structure simple and focused

## AI and Tooling Integration

- Leverage GitHub Copilot for code generation
- Use modern .NET tooling (dotnet CLI, Hot Reload, etc.)
- Reference .NET 10 features and capabilities
- Consider cloud-native and container deployment scenarios

## Migration Guidance

When providing migration examples:
- Show side-by-side comparisons (.NET 4.5 vs .NET 10)
- Explain deprecations and their modern alternatives
- Include practical migration steps
- Highlight breaking changes and workarounds

## References

- [Microsoft Learn - .NET](https://learn.microsoft.com/dotnet)
- [C# Language Specification](https://learn.microsoft.com/dotnet/csharp/)
- [ASP.NET Core Documentation](https://learn.microsoft.com/aspnet/core/)
- [.NET Blog](https://devblogs.microsoft.com/dotnet/)
