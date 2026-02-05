Aqui vão **exemplos avançados** de **Primary Constructors** no C# 12+ (disponível desde .NET 8). Eles vão além do uso básico (como em classes simples), mostrando cenários reais, boas práticas, armadilhas comuns, integração com DI (Dependency Injection), validação, herança, structs, generics e mais.

### 1. Injeção de Dependências (DI) em Classes de Serviço
Primary constructors são perfeitos para serviços com dependências injetadas.

```csharp
// Exemplo: Serviço com ILogger e repositório injetados
public class UsuarioService(
    ILogger<UsuarioService> logger,
    IUsuarioRepository repository)
{
    public async Task<Usuario?> ObterPorIdAsync(int id)
    {
        logger.LogInformation("Buscando usuário com ID {Id}", id);
        return await repository.ObterPorIdAsync(id);
    }

    public async Task AdicionarAsync(Usuario usuario)
    {
        if (usuario == null) throw new ArgumentNullException(nameof(usuario));
        await repository.AdicionarAsync(usuario);
        logger.LogInformation("Usuário {Nome} adicionado", usuario.Nome);
    }
}
```

**Por que avançado?**  
- Os parâmetros ficam disponíveis em todo o corpo da classe (métodos, propriedades, etc.).  
- Integra perfeitamente com o container de DI do ASP.NET Core (AddScoped<UsuarioService>()).

### 2. Validação no Construtor Primário + Captura de Parâmetros
Usando `ArgumentException` ou `ArgumentNullException` diretamente nos parâmetros.

```csharp
public class Produto(
    string nome,
    decimal preco,
    int estoqueMinimo = 10)
{
    // Validação inline (captura automática dos parâmetros)
    public string Nome { get; } = !string.IsNullOrWhiteSpace(nome)
        ? nome.Trim()
        : throw new ArgumentException("Nome não pode ser vazio", nameof(nome));

    public decimal Preco { get; } = preco >= 0
        ? preco
        : throw new ArgumentOutOfRangeException(nameof(preco), "Preço não pode ser negativo");

    public int EstoqueMinimo { get; } = estoqueMinimo;

    public bool PrecisaReporEstoque(int estoqueAtual) =>
        estoqueAtual < EstoqueMinimo;
}
```

**Dica avançada**: Use `nameof` para mensagens de erro claras. Isso é comum em DDD (Domain-Driven Design) para entidades imutáveis.

### 3. Primary Constructor em Structs (Performance Crítica)
Structs com primary constructors são ótimos para tipos valor (value types) de alta performance.

```csharp
public readonly struct Ponto3D(double x, double y, double z)
{
    public double X => x;
    public double Y => y;
    public double Z => z;

    public double DistanciaPara(Ponto3D outro) =>
        Math.Sqrt(
            Math.Pow(X - outro.X, 2) +
            Math.Pow(Y - outro.Y, 2) +
            Math.Pow(Z - outro.Z, 2));

    // Operador sobrecarregado
    public static Ponto3D operator +(Ponto3D a, Ponto3D b) =>
        new(a.X + b.X, a.Y + b.Y, a.Z + b.Z);
}
```

**Vantagem**: Sem boxing, sem alocação heap desnecessária. Ideal para jogos, computação científica ou ML.

### 4. Herança com Primary Constructors
Classes derivadas podem chamar o construtor base passando os parâmetros.

```csharp
public abstract class Animal(string nome, int idade)
{
    public string Nome { get; } = nome;
    public int Idade { get; } = idade;

    public abstract string Som();
}

public class Cachorro(string nome, int idade, string raca) 
    : Animal(nome, idade)
{
    public string Raca { get; } = raca;

    public override string Som() => "Au au!";
}

// Uso
var rex = new Cachorro("Rex", 5, "Labrador");
```

**Armadilha comum**: Se a classe derivada tiver seu próprio primary constructor, ela **deve** chamar `: base(...)` explicitamente, senão erro de compilação.

### 5. Primary Constructor com Generics e Constraints
```csharp
public class RepositorioCacheado<TEntidade, TChave>(IMemoriaCache cache)
    where TEntidade : class, IEntidade<TChave>
    where TChave : notnull
{
    private readonly IMemoriaCache _cache = cache;

    public async Task<TEntidade?> ObterPorChaveAsync(TChave chave)
    {
        if (!_cache.TryGetValue(chave, out TEntidade? entidade))
        {
            // Busca no banco...
            entidade = await BuscarNoBancoAsync(chave);
            _cache.Set(chave, entidade, TimeSpan.FromMinutes(10));
        }
        return entidade;
    }
}
```

**Avançado**: O parâmetro `cache` é capturado e usado em métodos. Muito útil em repositórios com caching.

### 6. Combinando com Records (Mistura de Imutabilidade)
Primary constructors funcionam bem com records (que já tinham isso antes).

```csharp
public record Pedido(
    Guid Id,
    Cliente Cliente,
    IReadOnlyList<ItemPedido> Itens,
    DateTime DataCriacao = default)
{
    public decimal ValorTotal => Itens.Sum(i => i.Valor);

    // Método com lógica
    public bool PodeCancelar() => DataCriacao > DateTime.UtcNow.AddHours(-24);
}
```

### 7. Armadilhas e Boas Práticas Avançadas
- **Não abuse**: Primary constructors são ótimos para classes pequenas/médias. Para classes grandes (> 5-6 parâmetros), prefira construtor explícito com comentários.
- **Campos backing**: Não crie campos privados duplicados. Use os parâmetros capturados diretamente.
- **Campos mutáveis**: Se precisar de estado mutável, crie propriedades com setter:
  ```csharp
  public class Contador(int inicial = 0)
  {
      public int Valor { get; private set; } = inicial;
      public void Incrementar() => Valor++;
  }
  ```
- **Atributos nos parâmetros**: Funciona!
  ```csharp
  public class Usuario([Required] string nome, [Range(18, 120)] int idade) { ... }
  ```
- **Debugging**: No debugger, os parâmetros aparecem como campos locais na classe.

Esses exemplos cobrem os casos mais avançados e produtivos na prática (API, DDD, performance, DI). Se quiser exemplos integrados com ASP.NET Core, Minimal APIs ou algo específico (ex: com Source Generators), é só pedir!
