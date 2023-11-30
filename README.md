# Fundamentos do Blazor com .NET 8

Olá Dev! 😎

Esse projeto faz parte do curso do [balta.io][balta].

O Blazor é a nova solução da Microsoft para aplicações Frontend.

## CONCEITOS

### SSG, SSR e SPA

1. SSG - Static Site Generation. Geração de site estático, Html, Javascript e CSS. Não há interação com nada.

   A vantagem de usar uma pagina SSG, é a velocidade para servir devido ao cache do navegador.

   Uma pagina SSG pode ser utilizada com o serviço Azure Static Web Apps, que possui baixo custo.
   Não necessidade de nenhum framework para rodar esse tipo de pagina.

   Exemplo: Site para artigos, blog, documentação, etc.

   Esse tipo de pagina é excelente para SEO (Indexador de sites de busca), ficando mais relevante em pesquisas.

1. SSR - Server Side Rendering. Renderização do lado do Servidor. As paginas estão sendo geradas no servidor por solicitação.

   A cada requisição é gerada uma pagina. Então temos aqui combinada a vantagem do SSG com interações e processamento do lado do servidor.

1. SPA - Single Page Application. Aplicação de página única. Esse [Exemplo][Passpad] carrega a aplicação uma vez e altera somente o necessário.

   Só é necessário alterar o "miolo" da pagina. Toda estrutura é a mesma, sendo efetivamente performático.

### Diferenças entre ASP.NET e Blazor

Ambos usam .NET Framework. O Blazor roda em cima do ASP.NET.
A maior diferença é a estrutura dos projetos. Enquanto o ASP.NET trabalha com uma arquitetura mais robusta, usando MVC, o Blazor trabalha com estrutura de páginas.

Qualquer aplicação que você criar com ASP.NET, você terá apenas a opção do SSR. Para interagir com o lado cliente, será necessário utilizar Javascript.

Já no caso do Blazor, temos ambas as opções, interativo com o lado do cliente e do servidor.

### Blazor WebAssembly

Projeto rodando diretamente via bytecode no Browser. O WebAssembly é muito mais performático que o Javascript.

Os WebAssembly podem rodar PWAs - Progressive Web App. Essas aplicações podem rodar tanto no navegador quanto no Desktop.

No Desktop, temos acesso a notificações e todos os recursos da máquina do usuário.

O PWA pode rodar offline, é apenas um pacote. Pode ser util para rodar em um servidor por ser muito simples.

### Templates

Blazor Server e Blazor WASM são tipos de projetos usando a versão .NET 7.

Na versão mais nova do .NET 8+, temos o template Blazor Web App. Na criação podemos selecionar o modo de interatividade.

### Enhanced Navigation e Form

A partir do Blazor 8, temos a transição entre o SSR e SPA, com otimização de SEO. Só é alterado o miolo da página.

Tudo é feito **Automagicamente**, bastando criar um projeto com esse novo template.

### Stream Rendering

Utilizando esse atributo, o carregamento é feito da forma mais rápida possível, trazendo a pagina em partes até o loading total.

Isso deixa a aplicação mais dinâmica e o usuário nao fica com a sensação de tela travada, clicando em algum componente diversas vezes para "forçar" a exibição dos dados.

### Render Modes

Os componentes precisam ter especificados qual será o modo de renderização. Caso não seja informado, o padrão é usar apenas via servidor, sendo o SSR. Os 3 modos de interatividade: Server, WebAssembly e Auto.

**Server** - Inicia mais rápido, porem sempre conectado ao servidor.
**WebAssembly** - Primeira execução lenta, precisa baixar o runtime. Próximas execuções muito rápidas.
**Auto** - Executa a primeira vez mais rápido, enquanto baixa o runtime, sendo o cenário mais inteligente.

### Blazor Hybrid

Une o Blazor com MAUI (Aplicação Mobile Multiplataforma). Podemos acessar recursos nativos do S.O. como alertas, camera, lanterna, etc. Para publicar um PWA, voce pode usar o [PWA Builder][PwaBuilder] e disponibilizar o App nas lojas de aplicativos, até mesmo rodar no Xbox.

### Blazor Identity

Sistema de login completo. O Identity foi portado do ASP.NET e facilita e muito a parte de segurança na aplicação.

Para criar um projeto com Identity, utilize:

```csharp
dotnet new blazor -o BlazorShop --auth Individual
```

## PROJETO BLAZOR SERVER - BLAZOR SHOP

1. Inicie um novo projeto com a instrução **dotnet new blazor -o BlazorShop --auth Individual**

   Passos no desenvolvimento orientado a dados - Data Driven Design (DDD):

   - **Models** -> Comece modelando as entidades conforme o contexto da aplicação.
   - **Data** -> Aplique o contexto ao banco de dados e reflita na aplicação a estrutura de campos e tabelas.
   - **Pages** -> Crie as paginas e componentes para visualizar e interagir com os dados.
   - **Pages** -> Adicione as validações antes do envio dos formulários e defina a segurança das rotas.

### Models

1. Crie uma pasta Models para modelagem das Entidades.

   ```csharp
   namespace BlazorShop.Models;

   public class Category
   {
       public int Id { get; set; }
       public string Title { get; set; } = string.Empty;
   }

   public class Product
   {
       public int Id { get; set; }
       public string Title { get; set; } = string.Empty;
       public string? Description { get; set; }
       public decimal Price { get; set; }
       public int CategoryId { get; set; }
       public Category Category { get; set; } = null!;
   }
   ```

2. Com as Models criadas, é hora de fazer as validações, vamos usar o **Data Annotations**, que é mais orientado a dados.
   Desenvolvimento Orientado a Dados ou DDD - Data Driven Design.

   ```csharp
   public class Category
   {
       // Indica o campo chave
       [Key]
       public int Id { get; set; }

       // Torna obrigatorio o preenchimento da Propriedade
       [Required(ErrorMessage = "Informe o título")]
       [MinLength(3, ErrorMessage = "O categoria deve ter pelo menos 3 caracteres")] // tamanho mínimo
       [MaxLength(60, ErrorMessage = "O categoria deve no máximo 60 caracteres")] // tamanho máximo
       public string Title { get; set; } = string.Empty;
   }

   public class Product
   {
       [Key] public int Id { get; set; }

       [Required(ErrorMessage = "Informe o título")]
       [MinLength(3, ErrorMessage = "O categoria deve ter pelo menos 3 caracteres")]
       [MaxLength(120, ErrorMessage = "O categoria deve no máximo 120 caracteres")]
       public string Title { get; set; } = string.Empty;

       public string? Description { get; set; }

       [Required(ErrorMessage = "Informe o preço do produto")]
       [DataType(DataType.Currency)] // Informa o tipo de dados no padrão Moeda R$
       public decimal Price { get; set; }

       public int CategoryId { get; set; }
       public Category Category { get; set; } = null!;
   }
   ```

   Com isso, temos o mapeamento para o banco de dados, seguindo esse tipo de abordagem.

### Data Context e Migrations

1. Agora, é a hora de aplicar o mapeamento ao banco de dados.

```csharp
using BlazorShop.Models;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace BlazorShop.Data;

public class ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : IdentityDbContext<ApplicationUser>(options)
{
    // Passando a referencia para criar as tabelas Produto e Categoria
    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<Category> Categories { get; set; } = null!;
}
```

1. Execute a migração do banco. Alterações no modelo da aplicação devem ser refletidas no banco de dados.

```csharp
// Como a geração do projeto executa uma migração v1 para o identity, adicione uma migração v2
dotnet ef migrations add v2

// Aplica a migration no banco de dados
dotnet ef update database
```

### Páginas e Rotas

1. Atualize o componente de layout NavMenu.razor, adicionando um link para navegação para categorias.

   ```csharp
   @implements IDisposable
   @inject NavigationManager NavigationManager

   // ... Demais códigos anteriores
     <div class="nav-item px-3">
         <NavLink class="nav-link" href="categories">
             <span class="bi bi-list-nested" aria-hidden="true"></span> Categorias
         </NavLink>
     </div>
   </div>
   // ... Demais códigos abaixo
   ```

   O projeto Blazor Server é ideal para executar comunicação com banco de dados. Se fosse um projeto Blazor WASM, os dados sensíveis como a conexão e segredos da aplicação ficariam expostos, pois roda diretamente do lado do cliente.

### Categorias

1. Faça um CRUD para categorias. Crie uma pasta Categories e adicione uma pagina Create.razor

   ```csharp
   @page "/categories/create" // Rota
   @inject ApplicationDbContext Context // Injeta o DbContext para interagir com o Banco de dados
   @inject NavigationManager Navigation // Injeta o Navigation, permitindo a navegação ou redirecionamento entre paginas
   @rendermode InteractiveServer // Renderização do lado do servidor
   @attribute [Authorize] // O usuário deve estar autenticado e possuir autorização para usar essa pagina

   <h1>Nova categoria</h1>

   // Feito vinculo no formulário da categoria ou também conhecido como Two way binding
   <EditForm Model="@Model" OnValidSubmit="OnValidSubmitAsync">

       // Chama as validações declaradas na Entidade Category
       <DataAnnotationsValidator />

       // Exibir na tela o resumo das validações
       <ValidationSummary />

       <div class="mb-3">
           <label class="form-label">Nome da categoria</label>

           // Recupera a propriedade do Titulo da Categoria
           <InputText @bind-Value="Model.Title" class="form-control"/>

           // Exibe apenas a validação especifica
           <ValidationMessage For="@(() => Model.Title)"/>
       </div>

       <button type="submit" class="btn btn-primary">
           Criar
       </button>
       <a href="/categories">Cancelar</a>
   </EditForm>

   @code {
       // Objeto vinculado ao formulário
       public Category Model { get; set; } = new();

       protected override void OnInitialized()
       {
       }

       // Ao enviar o formulário, grava o objeto no banco e redireciona para pagina categories
       public async Task OnValidSubmitAsync()
       {
           await Context.Categories.AddAsync(Model);
           await Context.SaveChangesAsync();
           Navigation.NavigateTo("categories");
       }
   }
   ```

2. Como temos a pagina que permite criar categorias, crie a pagina Index para listar as mesmas.

   ```csharp
   @page "/categories"
   @inject ApplicationDbContext Context
   @attribute [StreamRendering(true)] // Permite o carregamento por partes da pagina, melhorando a exp. do usuário.

   <h1>Categorias</h1>
   <a href="/categories/create" class="btn btn-primary">Nova Categoria</a>
   <br>

   // Usando o StreamRendering, exibe mensagem de aguardando o carregamento dos dados.
   @if (!Categories.Any())
   {
       <p class="text-center">
           <em>Carregando as categorias...</em>
       </p>
   }
   else
   { // Dados carregados, desenha tabela na tela
       <table class="table">
           <thead>
               <tr>
                   <th>Id</th>
                   <th>Nome</th>
                   <th></th>
               </tr>
           </thead>
           <tbody>
               @foreach (var category in Categories)
               {
                   <tr>
                       <td>@category.Id</td>
                       <td>
                           <a href="/categories/@category.Id">
                               @category.Title
                           </a>
                       </td>
                       <td> // Rota para editar a categoria
                           <a href="/categories/edit/@category.Id" class="btn btn-primary">
                               Editar
                           </a>
                           &nbsp;&nbsp;

                           // Função comentada no blazor usando @* Comentários... *@
                           // Somente é exibida função excluir para usuários com perfil "admin"
                           @* <AuthorizeView>
                               @if (context.User.IsInRole("admin"))
                               {
                                   <a href="/categories/delete/@category.Id" class="btn btn-danger">
                                       Excluir
                                   </a>
                               }
                           </AuthorizeView> *@

                           <a href="/categories/delete/@category.Id" class="btn btn-danger">
                               Excluir
                           </a>
                       </td>
                   </tr>
               }
           </tbody>
       </table>
   }

   @code {

       // Propriedade inicializando uma lista vazia de categorias
       public IEnumerable<Category> Categories { get; set; }
           = Enumerable.Empty<Category>();

       // Ao carregar a pagina, recupera as categorias e preenche a lista
       protected override async Task OnInitializedAsync()
       {
           Categories = await Context
               .Categories
               .AsNoTracking()
               .ToListAsync();
       }
   }
   ```

3. Crie a pagina Edit.razor para edição das categorias.

   ```csharp
   @page "/categories/edit/{id:int}" // Restrição de rota com parâmetro id que deve ser um numero inteiro
   @inject ApplicationDbContext Context
   @inject NavigationManager Navigation
   @rendermode InteractiveServer // Indica o processamento no servidor
   @attribute [Authorize] // Segurança no acesso a rota

   // Verifica o se o id passado como parâmetro existe
   @if (Model is null)
   {
       <p class="text-center">
           <em>Categoria não encontrada</em>
       </p>
   }
   else
   {
       // Titulo da pagina recuperando as propriedades da Model
       <h1>@Model.Title (@Model.Id)</h1>

       // Formulário para editar a categoria, similar ao formulário da pagina Create
       <EditForm Model="@Model" OnValidSubmit="OnValidSubmit">
           <DataAnnotationsValidator/>
           <ValidationSummary/>

           <div class="mb-3">
               <label class="form-label">Nome da categoria</label>
               <InputText @bind-Value="Model.Title" class="form-control"/>
               <ValidationMessage For="@(() => Model.Title)"/>
           </div>

           <button type="submit" class="btn btn-primary">
               Atualizar
           </button>
           <a href="/categories">Cancelar</a>
       </EditForm>
   }

   @code {

       // Indica que o valor da Propriedade Id será recuperado pela URL como Parâmetro
       [Parameter]
       public int Id { get; set; }

       public Category? Model { get; set; }

       // Carrega a categoria ao inicializar a pagina
       protected override async Task OnInitializedAsync()
       {
           Model = await Context
               .Categories
               .AsNoTracking()
               .FirstOrDefaultAsync(x => x.Id == Id);
       }

       // Ao enviar o formulário, salva as alterações no banco de dados
       public async Task OnValidSubmit()
       {
           Context.Categories.Update(Model);
           await Context.SaveChangesAsync();
           Navigation.NavigateTo("categories");
       }
   }
   ```

4. Crie a pagina Details.razor para exibir as informações da categoria.

   ```csharp
   @page "/categories/{id:int}"
   @inject ApplicationDbContext Context
   @rendermode InteractiveServer

   @if (Model is null)
   {
       <p class="text-center">
           <em>Categoria não encontrada</em>
       </p>
   }
   else
   {
       <h1>@Model.Title (@Model.Id)</h1>
       <EditForm Model="@Model">

           <div class="mb-3">
               <label class="form-label">Nome da categoria</label>
               <InputText @bind-Value="Model.Title" class="form-control" readonly/> // Somente leitura
           </div>

           <a href="/categories">Voltar</a>
       </EditForm>
   }

   @code {

       [Parameter]
       public int Id { get; set; }

       public Category? Model { get; set; }

       protected override async Task OnInitializedAsync()
       {
           Model = await Context
               .Categories
               .AsNoTracking()
               .FirstOrDefaultAsync(x => x.Id == Id);
       }
   }
   ```

5. Crie a pagina Delete.razor para excluir a categoria.

```csharp
@page "/categories/delete/{id:int}"
@inject ApplicationDbContext Context
@inject NavigationManager Navigation
@rendermode InteractiveServer
@* @attribute [Authorize(Roles = "ADMIN")] *@ // Apenas usuários com perfil ADMIN podem acessar a pagina

@if (Model is null)
{
    <p class="text-center">
        <em>Categoria não encontrada</em>
    </p>
}
else
{
    <h1>@Model.Title (@Model.Id)</h1>
    <EditForm Model="@Model" OnValidSubmit="OnValidSubmit">
        <DataAnnotationsValidator />
        <ValidationSummary />

        <div class="mb-3">
            <label class="form-label">Nome da categoria</label>
            <InputText @bind-Value="Model.Title" class="form-control" readonly />
            <ValidationMessage For="@(() => Model.Title)" />
        </div>

        <button type="submit" class="btn btn-danger">
            Excluir
        </button>
        <a href="/categories">Cancelar</a>
    </EditForm>
}

@code {

    [Parameter]
    public int Id { get; set; }

    public Category? Model { get; set; }

    protected override async Task OnInitializedAsync()
    {
        Model = await Context
            .Categories
            .AsNoTracking()
            .FirstOrDefaultAsync(x => x.Id == Id);
    }

    public async Task OnValidSubmit()
    {
        Context.Categories.Remove(Model);
        await Context.SaveChangesAsync();
        Navigation.NavigateTo("categories");
    }

}
```

### Produtos

A estrutura será similar ao CRUD de categorias.

1. Adicione a rota ao NavMenu.razor para acessar os produtos

   ```csharp
   // ... Códigos acima
   <div class="nav-item px-3">
      <NavLink class="nav-link" href="categories">
          <span class="bi bi-list-nested" aria-hidden="true"></span> Categorias
      </NavLink>
   </div>

   <div class="nav-item px-3">
      <NavLink class="nav-link" href="products">
          <span class="bi bi-list-nested" aria-hidden="true"></span> Produtos
      </NavLink>
   </div>
   // ... Códigos abaixo
   ```

1. Crie a pagina Create.razor para criar um produto.

   ```csharp
   @page "/products/create"
   @inject ApplicationDbContext Context
   @inject NavigationManager Navigation
   @rendermode InteractiveServer
   @attribute [Authorize]

   <h1>Novo Produto</h1>
   <EditForm Model="@Model" OnValidSubmit="OnValidSubmitAsync">
       <DataAnnotationsValidator />
       <ValidationSummary />

       <div class="mb-3">
           <label class="form-label">Nome do Produto</label>
           <InputText @bind-Value="Model.Title" class="form-control" />
           <ValidationMessage For="@(() => Model.Title)" />
       </div>

       <div class="mb-3">
           <label class="form-label">Preço</label>

           // Aqui é usado o InputNumber, pois estamos trabalhando com números.
           // Existem outros tipos de inputs para usar com Blazor, use o mais adequado de acordo
           // com o tipo de dados do campo
           <InputNumber @bind-Value="Model.Price" class="form-control" />
           <ValidationMessage For="@(() => Model.Price)" />
       </div>

       <div class="mb-3">
           <label class="form-label">Categoria</label>

           // InputSelect para listar e selecionar os dados em um Dropdown.
           // Vincula a propriedade CategoryId da Model Produto
           <InputSelect @bind-Value="Model.CategoryId" class="form-control">
               // Lista todas as categorias mostrando o titulo de cada uma, passando como valor
               // o Id da Categoria
               @foreach (var category in Categories)
               {
                   <option value="@category.Id">
                       @category.Title
                   </option>
               }
           </InputSelect>
       </div>

       <button type="submit" class="btn btn-primary">
           Criar
       </button>
       <a href="/categories">Cancelar</a>
   </EditForm>

   @code {
       // Cria os objetos para representar um Produto e uma lista de Categorias
       public Product Model { get; set; } = new();
       public IEnumerable<Category> Categories { get; set; } = Enumerable.Empty<Category>();

       // Ao carregar a pagina, carrega preenche a lista de categorias
       protected override async void OnInitialized()
       {
           Categories = await Context
               .Categories
               .AsNoTracking()
               .OrderBy(x => x.Title)
               .ToListAsync();
       }

       // Ao enviar o formulário, grava o produto
       public async Task OnValidSubmitAsync()
       {
           await Context.Products.AddAsync(Model);
           await Context.SaveChangesAsync();
           Navigation.NavigateTo("products");
       }
   }
   ```

1. Crie a pagina Index.razor, exibindo todos os produtos.

   ```csharp
   @page "/products"
   @inject ApplicationDbContext Context
   @attribute [StreamRendering(true)]

   <h1>Produtos</h1>
   <a href="/products/create" class="btn btn-primary">Novo Produto</a>
   <br>

   @if (!Products.Any())
   {
       <p class="text-center">
           <em>Carregando os produtos...</em>
       </p>
   }
   else
   {
       <table class="table">
           <thead>
               <tr>
                   <th>Id</th>
                   <th>Nome</th>
                   <th>Categoria</th>
                   <th>Preço</th>
                   <th></th>
               </tr>
           </thead>
           <tbody>
               @foreach (var product in Products)
               {
                   <tr>
                       <td>@product.Id</td>
                       <td>
                           <a href="/products/@product.Id">
                               @product.Title
                           </a>
                       </td>
                       <td>
                           @product.Category.Title
                       </td>
                       <td>
                           @product.Price.ToString("C", new CultureInfo("pt-BR"))
                       </td>
                       <td>
                           <a href="/products/edit/@product.Id" class="btn btn-primary">
                               Editar
                           </a>
                           &nbsp;&nbsp;
                           <a href="/products/delete/@product.Id" class="btn btn-danger">
                               Excluir
                           </a>
                       </td>
                   </tr>
               }
           </tbody>
       </table>
   }

   @code {

       public IEnumerable<Product> Products { get; set; }
           = Enumerable.Empty<Product>();

       protected override async Task OnInitializedAsync()
       {
           Products = await Context
               .Products
               .AsNoTracking()
               .Include(x => x.Category) // Recupera todas as propriedades da categoria
               .ToListAsync();
       }
   }
   ```

1. Crie a pagina Edit.razor para editar o produto.

```csharp
@page "/products/edit/{id:int}" // Id do produto passado via parâmetro na URL
@inject ApplicationDbContext Context
@inject NavigationManager Navigation
@rendermode InteractiveServer
@attribute [Authorize]

<h1>@Model.Title</h1>
<EditForm Model="@Model" OnValidSubmit="OnValidSubmitAsync">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <div class="mb-3">
        <label class="form-label">Nome do Produto</label>
        <InputText @bind-Value="Model.Title" class="form-control" />
        <ValidationMessage For="@(() => Model.Title)" />
    </div>

    <div class="mb-3">
        <label class="form-label">Preço</label>
        <InputNumber @bind-Value="Model.Price" class="form-control" />
        <ValidationMessage For="@(() => Model.Price)" />
    </div>

    <div class="mb-3">
        <label class="form-label">Categoria</label>
        <InputSelect @bind-Value="Model.CategoryId" class="form-control">
            @foreach (var category in Categories)
            {
                <option value="@category.Id">
                    @category.Title
                </option>
            }
        </InputSelect>
    </div>

    <button type="submit" class="btn btn-primary">
        Salvar
    </button>
    <a href="/categories">Cancelar</a>
</EditForm>

@code {

    // Id recuperado via URL
    [Parameter]
    public int Id { get; set; }

    public Product Model { get; set; } = new();
    public IEnumerable<Category> Categories { get; set; } = Enumerable.Empty<Category>();

    protected override async void OnInitialized()
    {
        // Se não encontrar o produto com o Id informado, cria um novo
        Model = await Context
            .Products
            .AsNoTracking()
            .FirstOrDefaultAsync(x => x.Id == Id) ?? new();

        // Recupera todas as categorias ordenando por título
        Categories = await Context
            .Categories
            .AsNoTracking()
            .OrderBy(x => x.Title)
            .ToListAsync();
    }

    // Ao enviar o formulário, salva as alterações
    public async Task OnValidSubmitAsync()
    {
        Context.Products.Update(Model);
        await Context.SaveChangesAsync();
        Navigation.NavigateTo("products");
    }
}
```

### Autorização

Para usar o recurso de autorização do ASP.NET, acesse o banco de dados e insira novos registros, vinculando perfis e usuários. Obs: caso precise gerar um novo GUID, execute no terminal **New-Guid**.

```sql
-- Cria um novo perfil Admin
INSERT INTO "main"."AspNetRoles" ("Id", "Name", "NormalizedName", "ConcurrencyStamp")
VALUES ('8fbc2a14-7b1e-40b1-bec0-82e33d1b806b', 'admin', 'ADMIN', '8fbc2a14-7b1e-40b1-bec0-82e33d1b806b');

-- Vincula o Id do Usuário com o Id do Perfil.
-- Obs: Copie o seu Id de usuário gerado após se registrar na aplicação.
INSERT INTO "main"."AspNetUserRoles" ("UserId", "RoleId")
VALUES ('9086dee8-40f5-460c-9aeb-0fc541eb1525', '8fbc2a14-7b1e-40b1-bec0-82e33d1b806b');
```

Para recuperar as informações de um usuário logado na aplicação, use como exemplo a pagina Auth.razor

```csharp
@page "/auth"

@using Microsoft.AspNetCore.Authorization

@attribute [Authorize]

<PageTitle>Auth</PageTitle>

<h1>You are authenticated</h1>

<AuthorizeView>
    Hello @context.User.Identity?.Name!
    <br />
    admin: @context.User.IsInRole("admin")
</AuthorizeView>

// Limitando o acesso por pelo perfil, por pagina. Ex: Categories -> Delete.razor

@page "/categories/delete/{id:int}"
@inject ApplicationDbContext Context
@inject NavigationManager Navigation
@rendermode InteractiveServer
@attribute [Authorize(Roles = "admin")] // Somente o Role "admin" pode acessar essa pagina
```

### Por enquanto, é isso aí. Bons estudos e bons códigos! 👍

[balta]: https://github.com/balta-io/3002
[Passpad]: https://andrebaltieri.github.io/passpad/
[PwaBuilder]: https://www.pwabuilder.com/
