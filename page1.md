Aqui está o conteúdo completo em formato **Markdown** (.md) que você pode copiar e colar diretamente em um arquivo chamado `cheatsheet-csharp-dotnet-8-to-10.md` (ou usar em um editor como VS Code, Obsidian ou GitHub para visualizar bonito).

```markdown
# Cheatsheet C# e .NET (8 a 10) – Novas Features, Sintaxe e Palavras Reservadas/Contextuais

**Período coberto**: .NET 8 (nov/2023) → .NET 10 (nov/2025, LTS atual em 2026)  
**C# associado**: C# 12 (.NET 8) → C# 14 (.NET 10)  
**Foco**: Novas palavras reservadas/contextuais, sintaxe, features de linguagem e runtime principais.  
**Fontes oficiais**: Microsoft Learn (docs .NET/C# whats-new).

Não há novas **palavras reservadas estritas** (como `class`, `interface`) desde C# 12. As novidades são **contextuais** (ex: `field`, `required` em versões anteriores) e sintaxe expandida.

## C# 12 – .NET 8 (2023)

### Principais Features
- **Primary Constructors** (construtores primários em classes/structs normais)  
  Parâmetros do construtor ficam visíveis em todo o corpo da classe/struct.  
  **Aplicação**: Simplifica classes com injeção de dependências ou dados imutáveis.  
  ```csharp
  public class Pessoa(string nome, int idade)
  {
      public string Nome => nome; // acessível diretamente
      public void Imprimir() => Console.WriteLine($"{nome}, {idade} anos");
  }
  var p = new Pessoa("João", 30);
  ```

- **Collection Expressions** (sintaxe curta para coleções)  
  ```csharp
  int[] numeros = [1, 2, 3];                // array
  List<string> nomes = ["Ana", "Beto"];     // List<T>
  Span<int> span = [..numeros, 4, 5];       // spread com ..
  ```

- **ref readonly parameters**  
  Indica que o parâmetro é referência read-only (melhor que `ref` para performance sem mutação).  
  ```csharp
  void Processar(ref readonly Pessoa p) { /* só leitura */ }
  ```

- **Default lambda parameters**  
  ```csharp
  var soma = (int a, int b = 10) => a + b;
  ```

- **Inline arrays** (arrays fixos performáticos em structs)  
  ```csharp
  [InlineArray(10)]
  public struct Buffer { private int _element; }
  Buffer buf; buf[0] = 42; // uso como array normal
  ```

- **Experimental attribute**  
  ```csharp
  [Experimental("MinhaFeature")]
  public void MetodoExperimental() { }
  ```

Outras: Alias any type, interceptors (preview).

**Palavras novas/contextuais**: Nenhuma forte; uso de `[` para collections.

## C# 13 – .NET 9 (2024)

### Principais Features
- **params collections** (params em Span, Memory, coleções genéricas)  
  **Aplicação**: APIs de alto desempenho sem alocação de array temporário.  
  ```csharp
  void Log(params ReadOnlySpan<string> mensagens)
  {
      foreach (var m in mensagens) Console.WriteLine(m);
  }
  Log("Erro", "Crítico"); // sem array explícito
  ```

- **lock com System.Threading.Lock**  
  Novo tipo `Lock` mais performático; `lock (obj)` usa-o automaticamente.  
  **Aplicação**: Melhor sincronização em cenários de alta concorrência.

- **\e escape sequence** (para ESC char)  
  ```csharp
  Console.Write("\e[31mVermelho\e[0m"); // ANSI color
  ```

- **Partial properties/indexers**  
  ```csharp
  partial class C { public partial int Valor { get; set; } }
  partial class C { private int _v; public partial int Valor { get => _v; set => _v = value; } }
  ```

- **Implicit indexer access in object initializers (^)**  
  ```csharp
  var arr = new int[5];
  arr = new[] { [^1] = 99, [^2] = 88 }; // último e penúltimo
  ```

- **ref struct implementando interfaces** (limitado, sem boxing)  
- **allows ref struct constraint**  
  ```csharp
  class Gen<T> where T : allows ref struct { }
  ```

- **field** (contextual keyword, preview em 13, full em 14)  
  Acessa backing field sintetizado.  
  ```csharp
  public string Nome { get => field; set => field = value?.Trim(); }
  ```

Outras: Method group natural type improvements, ref locals em async/iterators.

## C# 14 – .NET 10 (2025)

### Principais Features
- **Extension Members** (nova sintaxe para extensions com properties, operators, statics)  
  **Aplicação**: Estender tipos de forma mais rica (LINQ-like).  
  ```csharp
  extension(IEnumerable<int> seq)
  {
      public int Sum() => seq.Sum(); // instance
      public static IEnumerable<int> Empty => Enumerable.Empty<int>(); // static
      public static IEnumerable<int> operator +(IEnumerable<int> a, IEnumerable<int> b) => a.Concat(b);
  }
  var soma = minhaLista.Sum(); // extension instance
  ```

- **field keyword** (full, contextual)  
  Backing field automático em props.  
  ```csharp
  public int Contador { get; set => field = value > 0 ? value : throw new ArgumentException(); }
  ```

- **Null-conditional assignment** (?.= e ?[]=)  
  ```csharp
  cliente?.Endereco = novoEndereco; // só atribui se cliente != null
  cliente?.Pedidos?[0] = pedido;
  ```

- **nameof unbound generics**  
  ```csharp
  Console.WriteLine(nameof(List<>)); // "List"
  ```

- **Simple lambda parameters with modifiers** (ref, in, out, scoped sem tipo explícito)  
  ```csharp
  var parse = (string s, out int i) => int.TryParse(s, out i);
  ```

- **More partial members** (partial constructors, events)  
- **User-defined compound assignment** (+=, -= etc. custom)  
- **Implicit Span conversions** (Span/ReadOnlySpan mais naturais)

**Palavras novas/contextuais**: `field` (contextual), sintaxe `extension (Type) { ... }`.

## .NET 10 Highlights (Runtime e Ecossistema)

- **Runtime**: Melhorias em JIT, AVX10.2, NativeAOT, loop inversion, stack allocs.  
- **ASP.NET Core**: Passkeys (WebAuthn), Blazor preloading + form validation, Minimal APIs refinadas, OpenAPI melhorado.  
- **Bibliotecas**: Async ZIP APIs, cryptography post-quantum, WebSocketStream simplificado.  
- **MAUI**: MediaPicker múltiplos arquivos, WebView interception, Android 15/16 support.  
- **SDK**: Microsoft.Testing.Platform em `dotnet test`, melhor container support, tab-completion CLI.  
- **EF Core**: LINQ otimizado, Cosmos DB melhorado.

**Dicas de uso prático**:
- Use **Native AOT** para apps cloud/console rápidos.
- Prefira **Minimal APIs** + **Blazor Hybrid** para apps modernos.
- Ative **C# latest** no projeto: `<LangVersion>latest</LangVersion>`.

**Atualizado em fevereiro 2026** – .NET 10 é a LTS atual (suporte até ~2028).  
Para mais detalhes: https://learn.microsoft.com/en-us/dotnet/csharp/whats-new/
```

Copie todo o bloco acima para um arquivo .md. Se quiser expandir com mais exemplos ou focar em alguma área específica (ex: ASP.NET Core 10, MAUI 10), avise que eu adiciono!
