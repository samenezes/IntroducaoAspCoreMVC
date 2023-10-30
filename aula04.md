# Adicione um modelo a um aplicativo ASP.NET Core MVC

Neste tutorial, classes são adicionadas para o gerenciamento de filmes em um banco de dados.
Essas classes serão a parte do “Modelo” parte do aplicativo MVC.

Essas classes do modelo são usadas com o Entity Framework Core (EF Core) para trabalhar com um banco de dados. 
O EF Core é uma estrutura ORM (mapeamento relacional de objetos) que simplifica o código de acesso a dados que você precisa escrever.

As classes de modelo criadas são conhecidas como classes POCO , de Plain Old CLR Objects (Bons e velhos objetos do CLR).
As classes POCO não têm nenhuma dependência em EF Core. 
Elas definem as propriedades dos dados a serem armazenados no banco de dados.

Neste tutorial, você escreve as classes de modelo primeiro e o EF Core cria o banco de dados.

## Adicionar uma classe de modelo de dados

Clique com o botão direito do mouse na pasta Modelos>Adicionar>Classe. Dê o nome Movie.cs para o arquivo.

Atualize o arquivo Models/Movie.cs com o seguinte código:

~~~c #
using System.ComponentModel.DataAnnotations;

namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }
    public string? Title { get; set; }
    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }
    public string? Genre { get; set; }
    public decimal Price { get; set; }
}
~~~

 A classe Movie contém um campo Id, que é exigido pelo banco de dados para a chave primária.
 O atributo DataType em ReleaseDate especifica o tipo de dados (Date). Com esse atributo:
 O usuário não precisa inserir informações de horário no campo de data.
 Somente a data é exibida, não as informações de tempo.

O ponto de interrogação após string indica que a propriedade permite valor nulo.

## Adicionar pacotes do NuGet

O Visual Studio instala automaticamente os pacotes necessários.

Compile o projeto como uma verificação de erros do compilador.

## Aplicar scaffold a páginas de filme

Use a ferramenta scaffolding para produzir páginas CRUD (Create, Read, Update e Delete) para o modelo de filme.

Clique com o botão direito do mouse na pasta Controladores do Gerenciador de Soluções e selecione Adicionar > Novo Item Gerado por Scaffolding.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/da805019-d9b0-4d5b-b034-300378e6d4ae)

No diálogo Adicionar novo item de Scaffold:

No painel esquerdo, selecione Instalado>Comum>MVC.
Selecione Controlador MVC com exibições, usando o Entity Framework.
Selecione Adicionar.

## NA REDE DO COTEMIG NÃO IRÁ APARECER A OPÇÃO DE SQL, CONFORME IMAGEM ABAIXO, PORTANTO NÃO SE PREOCUPE EM SELECIONAR SQL SERVER, NÃO IRÁ DAR ERRO.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/b7475cf9-d1ce-4d1a-b10b-a26077cb24b1)

Complete a caixa de diálogo Adicionar Controlador MVC com exibições, usando o Entity Framework:

Na lista suspensa Classe de modelo, selecione Filme (MvcMovie.Models).
Na linha Classe de contexto de dados, selecione o sinal + (adição).
Na caixa de diálogo Adicionar Contexto de Dados , o nome da classe MvcMovie.Data.MvcMovieContext é gerado.
Selecione Adicionar.
Na lista suspensa Provedor de banco de dados, selecione SQL Server.
Exibições e Nome do controlador: mantenha o padrão.
Selecione Adicionar.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/2cd37efd-a5ea-4a24-af71-ae5d3159faa9)

Se você receber uma mensagem de erro, selecione Adicionar uma segunda vez para tentar novamente.


O scaffolding adiciona os seguintes pacotes:

Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools
Microsoft.VisualStudio.Web.CodeGeneration.Design
O scaffolding cria o seguinte:

Um controlador de filmes: Controllers/MoviesController.cs
Razor exibir arquivos para páginas Criar, Excluir, Detalhes, Editar e Índice : Views/Movies/*.cshtml
Uma classe de contexto de banco de dados: Data/MvcMovieContext.cs
O scaffolding atualiza o seguinte:

Insere as referências de pacote necessárias no arquivo de projeto MvcMovie.csproj.
Registra o contexto do banco de dados no arquivo Program.cs.
Adicionar uma cadeia de caracteres de conexão do banco de dados ao arquivo appsettings.json.
A criação automática desses arquivos e atualizações de arquivos é conhecida como scaffolding.

As páginas com scaffolding ainda não podem ser usadas porque o banco de dados não existe. 
Executar o aplicativo e selecionar o link Aplicativo de Filme resulta em uma mensagem de erro Não é possível abrir o banco de dados ou nenhuma tabela desse tipo: filme.

Crie o aplicativo para verificar se não há erros.

## Migração inicial

Use o recurso EF CoreMigrações para criar o banco de dados.
O recurso Migrações é um conjunto de ferramentas que cria e atualiza um banco de dados para corresponder ao modelo de dados.

No menu Ferramentas, selecione Gerenciador de Pacotes NuGet>Console do Gerenciador de Pacotes.

No PMC (Console do Gerenciador de Pacotes), Insira os seguintes comandos:

~~~ powershell #

Add-Migration InitialCreate
Update-Database

~~~

Add-Migration InitialCreate: gera um arquivo de migração Migrations/{timestamp}_InitialCreate.cs.
O argumento InitialCreate é o nome da migração. Qualquer nome pode ser usado, mas, por convenção, um nome que descreve a migração é selecionado. 
Como essa é a primeira migração, a classe gerada contém o código para criar o esquema de banco de dados. O esquema de banco de dados é baseado no modelo especificado na classe MvcMovieContext.

Update-Database: atualiza o banco de dados para a migração mais recente, que o comando anterior criou. Esse comando executa o método Up no arquivo Migrations/{time-stamp}_InitialCreate.cs, que cria o banco de dados.

O comando Update-Database gera o seguinte aviso:

Nenhum tipo foi especificado para a coluna decimal 'Preço' no tipo de entidade 'Filme'. Isso fará com que valores sejam truncados silenciosamente se não couberem na precisão e na escala padrão. Especifique explicitamente o tipo de coluna do SQL Server que pode acomodar todos os valores usando 'HasColumnType()'.

Ignore o aviso anterior, ele é corrigido em um tutorial posterior.

## Testar o aplicativo

Execute o aplicativo e clique no link Aplicativo de Filme.

Se você receber uma exceção semelhante à seguinte, talvez tenha perdido o comando Update-Database na etapa de migrações:

~~~ console #
SqlException: Cannot open database "MvcMovieContext-1" requested by the login. The login failed.
~~~

## Examinar a classe de contexto e o registro do banco de dados gerados

Com o EF Core, o acesso aos dados é executado usando um modelo. Um modelo é feito de classes de entidade e um objeto de contexto que representa uma sessão com o banco de dados.
O objeto de contexto permite consultar e salvar dados. O contexto de banco de dados é derivado de Microsoft. 
EntityFrameworkCore.DbContext e especifica as entidades a serem incluídas no modelo de dados.

O scaffolding cria a classe de contexto do banco de dados Data/MvcMovieContext.cs:

~~~ c #
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;
using MvcMovie.Models;

namespace MvcMovie.Data
{
    public class MvcMovieContext : DbContext
    {
        public MvcMovieContext (DbContextOptions<MvcMovieContext> options)
            : base(options)
        {
        }

        public DbSet<MvcMovie.Models.Movie> Movie { get; set; }
    }
}
~~~

O código anterior cria uma propriedade DbSet<Movie> que representa os filmes no banco de dados.

## Injeção de dependência

O ASP.NET Core foi criado com a DI (injeção de dependência). Serviços, como o contexto do banco de dados, são registrados com DI no Program.cs. 
Esses serviços são fornecidos aos componentes que necessitam deles através de parâmetros do construtor.

No arquivo Controllers/MoviesController.cs, o construtor usa a Injeção de Dependência para injetar o contexto do banco de dados MvcMovieContext no controlador.
O contexto de banco de dados é usado em cada um dos métodos CRUD no controlador.

O scaffolding gerou o seguinte código realçado em Program.cs:

~~~ c #
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<MvcMovieContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("MvcMovieContext")));
~~~

O sistema de configuração ASP.NET Core faz a leitura da cadeia de caracteres de conexão de banco de dados "MvcMovieContext".


## Examinar a cadeia de caracteres de conexão de banco de dados gerada

O scaffolding adicionou uma cadeia de conexão ao arquivo appsettings.json:

~~~ json #
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "MvcMovieContext": "Data Source=MvcMovieContext-ea7a4069-f366-4742-bd1c-3f753a804ce1.db"
  }
}
~~~

No desenvolvimento local, o sistema de configuração do ASP.NET Core lê a chave ConnectionString do arquivo appsettings.json.

## A classe InitialCreate

Examinar o arquivo de migração Migrations/{timestamp}_InitialCreate.cs:

~~~ c #
using System;
using Microsoft.EntityFrameworkCore.Migrations;

#nullable disable

namespace MvcMovie.Migrations
{
    public partial class InitialCreate : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.CreateTable(
                name: "Movie",
                columns: table => new
                {
                    Id = table.Column<int>(type: "int", nullable: false)
                        .Annotation("SqlServer:Identity", "1, 1"),
                    Title = table.Column<string>(type: "nvarchar(max)", nullable: true),
                    ReleaseDate = table.Column<DateTime>(type: "datetime2", nullable: false),
                    Genre = table.Column<string>(type: "nvarchar(max)", nullable: true),
                    Price = table.Column<decimal>(type: "decimal(18,2)", nullable: false)
                },
                constraints: table =>
                {
                    table.PrimaryKey("PK_Movie", x => x.Id);
                });
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropTable(
                name: "Movie");
        }
    }
}
~~~

No código anterior:

InitialCreate.Up cria a tabela de Filmes e configura Id como a chave primária.
O método InitialCreate.Down reverte as alterações de esquema feitas pela migração de Up.

## Injeção de dependência no controlador

Abra o arquivo Controllers/MoviesController.cs e examine o construtor:

~~~ c #
public class MoviesController : Controller
{
    private readonly MvcMovieContext _context;

    public MoviesController(MvcMovieContext context)
    {
        _context = context;
    }
~~~

O construtor usa a Injeção de Dependência para injetar o contexto de banco de dados (MvcMovieContext) no controlador.
O contexto de banco de dados é usado em cada um dos métodos CRUD no controlador.

Teste a página Criar. Inserir e enviar dados.

Teste os links Editar, Detalhes e Excluir.


Modelos fortemente tipados e a diretiva @model
Anteriormente neste tutorial, você viu como um controlador pode passar dados ou objetos para uma exibição usando o dicionário ViewData.
O dicionário ViewData é um objeto dinâmico que fornece uma maneira conveniente de associação tardia para passar informações para uma exibição.

O MVC também fornece a capacidade de passar objetos de modelo fortemente tipados para uma exibição. Essa abordagem fortemente tipada permite a verificação de código em tempo de compilação.
O mecanismo de scaffolding passou um modelo fortemente tipado na classe e nas exibições de MoviesController.

Examine o método de Details gerado no arquivo Controllers/MoviesController.cs :

~~~ c #
// GET: Movies/Details/5
public async Task<IActionResult> Details(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    var movie = await _context.Movie
        .FirstOrDefaultAsync(m => m.Id == id);
    if (movie == null)
    {
        return NotFound();
    }

    return View(movie);
}
~~~

O parâmetro id geralmente é passado como dados de rota. Por exemplo, https://localhost:5001/movies/details/1 define:

O controlador para o controlador movies, o primeiro segmento da URL.
A ação para details, o segundo segmento da URL.
O idpara 1, o último segmento da URL.
O id pode ser passado com uma cadeia de caracteres de consulta, como no exemplo a seguir:

https://localhost:5001/movies/details?id=1

O parâmetro id é definido como um tipo que permite valor nulo (int?) nos casos em que o valor de id não seja fornecido.

Um expressão lambda é passada para o método FirstOrDefaultAsync para selecionar as entidades de filmes que correspondem ao valor dos dados da rota ou da cadeia de consulta.

~~~ c #

var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
~~~

Se for encontrado um filme, uma instância do modelo Movie será passada para a exibição Details:

~~~ c #

return View(movie);
~~~

Examinar o conteúdo do arquivo Views/Movies/Details.cshtml:

~~~ cshtml #

@model MvcMovie.Models.Movie

@{
    ViewData["Title"] = "Details";
}

<h1>Details</h1>

<div>
    <h4>Movie</h4>
    <hr />
    <dl class="row">
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Title)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Title)
        </dd>
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.ReleaseDate)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.ReleaseDate)
        </dd>
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Genre)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Genre)
        </dd>
        <dt class = "col-sm-2">
            @Html.DisplayNameFor(model => model.Price)
        </dt>
        <dd class = "col-sm-10">
            @Html.DisplayFor(model => model.Price)
        </dd>
    </dl>
</div>
<div>
    <a asp-action="Edit" asp-route-id="@Model.Id">Edit</a> |
    <a asp-action="Index">Back to List</a>
</div>

~~~

A instrução @model na parte superior do arquivo de exibição especifica o tipo de objeto que a exibição espera. 
Quando o controlador de filme foi criado, a seguinte instrução @model foi incluída:

~~~ cshtml #

@model MvcMovie.Models.Movie

~~~

Essa diretiva @model permite o acesso ao filme que o controlador passou para a exibição. O objeto Model é fortemente tipado. 
Por exemplo, na exibição Details.cshtml, o código passa cada campo de filme para os Auxiliares de HTML DisplayNameFor e DisplayFor com o objeto fortemente tipado Model.
Os métodos Create e Edit e as exibições também passam um objeto de modelo Movie.

Examine a exibição Index.cshtml e o método Index no controlador Filmes. 
Observe como o código cria um objeto List quando ele chama o método View. 
O código passa esta lista Movies do método de ação Index para a exibição:

~~~ c #
// GET: Movies
public async Task<IActionResult> Index()
{
    return View(await _context.Movie.ToListAsync());
}
~~~

O código retornará os detalhes do problema se a propriedade Movie do contexto dos dados for nula.

Quando o controlador de filmes foi criado, o scaffolding incluiu a seguinte instrução @model na parte superior do arquivo Index.cshtml:

~~~ cshtml #
@model IEnumerable<MvcMovie.Models.Movie>
~~~

A diretiva @model permite acessar a lista de filmes que o controlador passou para a exibição usando um objeto Model fortemente tipado.
Por exemplo, na exibição Index.cshtml o código executa um loop pelos filmes com uma instrução foreach no objeto Model fortemente tipado:

~~~ cshtml #
@model IEnumerable<MvcMovie.Models.Movie>

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-action="Create">Create New</a>
</p>
<table class="table">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.Title)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.ReleaseDate)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Genre)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Price)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Title)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.ReleaseDate)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Genre)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Price)
            </td>
            <td>
                <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |
                <a asp-action="Details" asp-route-id="@item.Id">Details</a> |
                <a asp-action="Delete" asp-route-id="@item.Id">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>
~~~
Como o objeto Model é fortemente tipado como um objeto IEnumerable<Movie>, cada item no loop é tipado como Movie. 
Entre outros benefícios, o compilador valida os tipos usados no código.




