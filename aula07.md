# Adicione uma pesquisa ao aplicativo ASP.NET Core MVC

Nesta seção, você adiciona a funcionalidade de pesquisa ao método de ação Index que permite pesquisar filmes por gênero ou nome.

Atualize o método Index encontrado dentro de Controllers/MoviesController.cs com o seguinte código:

~~~ c #

public async Task<IActionResult> Index(string searchString)
{
    if (_context.Movie == null)
    {
        return Problem("Entity set 'MvcMovieContext.Movie'  is null.");
    }

    var movies = from m in _context.Movie
                select m;

    if (!String.IsNullOrEmpty(searchString))
    {
        movies = movies.Where(s => s.Title!.Contains(searchString));
    }

    return View(await movies.ToListAsync());
}

~~~

A linha a seguir do método de ação Index cria uma consulta LINQ para selecionar os filmes:

~~~ c #

var movies = from m in _context.Movie
             select m;

~~~

A consulta é definida apenas neste ponto; ela não foi executada no banco de dados.

Se o parâmetro searchString contiver uma cadeia de caracteres, a consulta de filmes será modificada para filtrar o valor da cadeia de caracteres de pesquisa:

~~~ c #
if (!String.IsNullOrEmpty(searchString))
{
    movies = movies.Where(s => s.Title!.Contains(searchString));
}
~~~

O código s => s.Title!.Contains(searchString) acima é uma Expressão Lambda.
Lambdas são usados em consultas LINQ baseadas em método como argumentos para métodos de operadores de consulta padrão, como o método Where ou Contains (usado no código acima).
As consultas LINQ não são executadas quando são definidas ou no momento em que são modificadas com uma chamada a um método, como Where, Contains ou OrderBy. 
Em vez disso, a execução da consulta é adiada. 
Isso significa que a avaliação de uma expressão é atrasada até que seu valor realizado seja, de fato, iterado ou o método ToListAsync seja chamado.

O método Contains é executado no banco de dados, não no código C# mostrado acima.
A diferenciação de maiúsculas e minúsculas na consulta depende do banco de dados e da ordenação. 
No SQL Server, Contains é mapeado para SQL LIKE, que não diferencia maiúsculas de minúsculas. 
No SQLite, com a ordenação padrão, ele diferencia maiúsculas de minúsculas.

Navegue até /Movies/Index. 

Acrescente uma cadeia de consulta, como ?searchString=Ghost, à URL.

Os filmes filtrados são exibidos.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/5e654f92-a3db-4fcd-9c7c-0bb380fddefb)

Se você alterar a assinatura do método Index para que ele tenha um parâmetro chamado id, o parâmetro id corresponderá o espaço reservado {id} opcional com as rotas padrão definidas em Program.cs.

~~~ c #

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}"); // essa linha

~~~

Altere o parâmetro para id e todas as ocorrências de searchString para id.

O método Index anterior:


~~~ c #
public async Task<IActionResult> Index(string searchString)
{
    if (_context.Movie == null)
    {
        return Problem("Entity set 'MvcMovieContext.Movie'  is null.");
    }

    var movies = from m in _context.Movie
                select m;

    if (!String.IsNullOrEmpty(searchString))
    {
        movies = movies.Where(s => s.Title!.Contains(searchString));
    }

    return View(await movies.ToListAsync());
}
~~~

O método Index atualizado com o parâmetro id:

~~~ c #
public async Task<IActionResult> Index(string id)   //essa linha
{
    if (_context.Movie == null)
    {
        return Problem("Entity set 'MvcMovieContext.Movie'  is null.");
    }

    var movies = from m in _context.Movie
                 select m;

    if (!String.IsNullOrEmpty(id))  //essa linha
    {
        movies = movies.Where(s => s.Title!.Contains(id)); //essa linha
    }

    return View(await movies.ToListAsync());
}
~~~

Agora você pode passar o título de pesquisa como dados de rota (um segmento de URL), em vez de como um valor de consulta.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/039c848d-3b24-4259-9612-485ad30bcb32)

No entanto, você não pode esperar que os usuários modifiquem a URL sempre que desejarem pesquisar um filme. 
Agora você adicionará os elementos da interface do usuário para ajudá-los a filtrar filmes. 
Se você tiver alterado a assinatura do método Index para testar como passar o parâmetro ID associado à rota, # altere-o novamente para que ele use um parâmetro chamado searchString:

~~~ c #

public async Task<IActionResult> Index(string searchString)  //essa linha
{
    if (_context.Movie == null)
    {
        return Problem("Entity set 'MvcMovieContext.Movie'  is null.");
    }

    var movies = from m in _context.Movie
                select m;

    if (!String.IsNullOrEmpty(searchString))  // essa linha
    {
        movies = movies.Where(s => s.Title!.Contains(searchString)); //essa linha
    }

    return View(await movies.ToListAsync());
}

~~~

Abra o arquivo Views/Movies/Index.cshtml e adicione a marcação <form> destacada abaixo:

~~~ cshtml #
@model IEnumerable<MvcMovie.Models.Movie>

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-action="Create">Create New</a>
</p>

<form asp-controller="Movies" asp-action="Index">             //essa linha
    <p>                                                       //essa linha
        Title: <input type="text" name="SearchString" />      //essa linha   
        <input type="submit" value="Filter" />                //essa linha
    </p>                                                      //essa linha
</form>                                                       //essa linha
<table class="table">

~~~

A marcação <form> HTML usa o Auxiliar de Marcação de Formulário. 
Portanto, quando você enviar o formulário, a cadeia de caracteres de filtro será enviada para a ação Index do controlador de filmes. 
Salve as alterações e, em seguida, teste o filtro.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/84b5531b-7a2b-496b-9e13-2e7b16b8a41d)

Não há nenhuma sobrecarga [HttpPost] do método Index que poderia ser esperada.
Isso não é necessário, porque o método não está alterando o estado do aplicativo, apenas filtrando os dados.

Você poderá adicionar o método [HttpPost] Index a seguir.

~~~ c #

[HttpPost]
public string Index(string searchString, bool notUsed) // essa linha
{
    return "From [HttpPost]Index: filter on " + searchString;
}
~~~

O parâmetro notUsed é usado para criar uma sobrecarga para o método Index.
Falaremos sobre isso mais adiante no tutorial.

Se você adicionar esse método, o chamador de ação fará uma correspondência com o método [HttpPost] Index e o método [HttpPost] 
Index será executado conforme mostrado na imagem abaixo.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/fdccc9e3-4f78-481c-a747-00f868d36ebd)

No entanto, mesmo se você adicionar esta versão [HttpPost] do método Index, haverá uma limitação na forma de como isso tudo foi implementado. 
Imagine que você deseja adicionar uma pesquisa específica como Favoritos ou enviar um link para seus amigos para que eles possam clicar para ver a mesma lista filtrada de filmes. 
Observe que a URL da solicitação HTTP POST é a mesma que a URL da solicitação GET (localhost:{PORT}/Movies/Index) – não há nenhuma informação de pesquisa na URL. 
As informações de cadeia de caracteres de pesquisa são enviadas ao servidor como um valor de campo de formulário. 
Verifique isso com as ferramentas do Desenvolvedor do navegador.
A imagem abaixo mostra as ferramentas do Desenvolvedor do navegador Chrome:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/6352e5bb-26b5-454e-bde7-7a031f94ef64)

Veja o parâmetro de pesquisa e o token XSRF no corpo da solicitação. Observe, conforme mencionado no tutorial anterior, que o Auxiliar de Marcação de Formulário gera um token antifalsificação XSRF. 
Não modificaremos os dados e, portanto, não precisamos validar o token no método do controlador.

Como o parâmetro de pesquisa está no corpo da solicitação e não na URL, não é possível capturar essas informações de pesquisa para adicionar como Favoritos ou compartilhar com outras pessoas.

Corrigimos isso especificando que a solicitação deve ser HTTP GET encontrada no arquivo Views/Movies/Index.cshtml.

~~~ c #
@model IEnumerable<MvcMovie.Models.Movie>

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-action="Create">Create New</a>
</p>

<form asp-controller="Movies" asp-action="Index" method="get">  //essa linha
    <p>
        Title: <input type="text" name="SearchString" />
        <input type="submit" value="Filter" />
    </p>
</form>
<table class="table">
~~~

Agora, quando você enviar uma pesquisa, a URL conterá a cadeia de consulta da pesquisa. 
A pesquisa também irá para o método de ação HttpGet Index, mesmo se você tiver um método HttpPost Index.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/c671e682-9014-4adf-bcdb-3128f3de52cd)

A seguinte marcação mostra a alteração para a marcação form:

~~~ cshtml #

<form asp-controller="Movies" asp-action="Index" method="get">

~~~

## Adicionar pesquisa por gênero

Adicione a seguinte classe MovieGenreViewModel à pasta Models:

~~~ c #

using Microsoft.AspNetCore.Mvc.Rendering;
using System.Collections.Generic;

namespace MvcMovie.Models;

public class MovieGenreViewModel
{
    public List<Movie>? Movies { get; set; }
    public SelectList? Genres { get; set; }
    public string? MovieGenre { get; set; }
    public string? SearchString { get; set; }
}
~~~

O modelo de exibição do gênero de filme conterá:

 1. Uma lista de filmes.
 2. Uma SelectList que contém a lista de gêneros. Isso permite que o usuário selecione um gênero na lista.
 3. MovieGenre, que contém o gênero selecionado.
 4. SearchString, que contém o texto que os usuários inserem na caixa de texto de pesquisa.
    
Substitua o método Index em MoviesController.cs pelo seguinte código:

~~~ c #

// GET: Movies
public async Task<IActionResult> Index(string movieGenre, string searchString)
{
    if (_context.Movie == null)
    {
        return Problem("Entity set 'MvcMovieContext.Movie'  is null.");
    }

    // Use LINQ to get list of genres.
    IQueryable<string> genreQuery = from m in _context.Movie
                                    orderby m.Genre
                                    select m.Genre;
    var movies = from m in _context.Movie
                 select m;

    if (!string.IsNullOrEmpty(searchString))
    {
        movies = movies.Where(s => s.Title!.Contains(searchString));
    }

    if (!string.IsNullOrEmpty(movieGenre))
    {
        movies = movies.Where(x => x.Genre == movieGenre);
    }

    var movieGenreVM = new MovieGenreViewModel
    {
        Genres = new SelectList(await genreQuery.Distinct().ToListAsync()),
        Movies = await movies.ToListAsync()
    };

    return View(movieGenreVM);
}
~~~

O código a seguir é uma consulta LINQ que recupera todos os gêneros do banco de dados.

~~~ c #

// Use LINQ to get list of genres.
IQueryable<string> genreQuery = from m in _context.Movie
                                orderby m.Genre
                                select m.Genre;
~~~

A SelectList de gêneros é criada com a projeção dos gêneros distintos (não desejamos que nossa lista de seleção tenha gêneros duplicados).

Quando o usuário pesquisa o item, o valor de pesquisa é mantido na caixa de pesquisa.

## Adicionar pesquisa por gênero à exibição Índice

Atualize Index.cshtml, encontrado em Views/Movies/, da seguinte maneira:

~~~ cshtml #

@model MvcMovie.Models.MovieGenreViewModel

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-action="Create">Create New</a>
</p>
<form asp-controller="Movies" asp-action="Index" method="get">
    <p>

        <select asp-for="MovieGenre" asp-items="Model.Genres">
            <option value="">All</option>
        </select>

        Title: <input type="text" asp-for="SearchString" />
        <input type="submit" value="Filter" />
    </p>
</form>

<table class="table">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.Movies![0].Title)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Movies![0].ReleaseDate)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Movies![0].Genre)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Movies![0].Price)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model.Movies!)
        {
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

Examine a expressão lambda usada no auxiliar HTML a seguir:

@Html.DisplayNameFor(model => model.Movies![0].Title)

No código anterior, o Auxiliar de HTML DisplayNameFor inspeciona a propriedade Title referenciada na expressão lambda para determinar o nome de exibição. 
Como a expressão lambda é inspecionada em vez de avaliada, você não recebe uma violação de acesso quando model, model.Movies ou model.Movies[0] é null ou vazio.
Quando a expressão lambda é avaliada (por exemplo, @Html.DisplayFor(modelItem => item.Title)), os valores da propriedade do modelo são avaliados.
O ! após model.Movies é o operador que permite valor nulo, que é usado para declarar que Movies não é nulo.

Teste o aplicativo pesquisando por gênero, título do filme e por ambos:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/0c20e121-4658-4c98-a0f9-cb66afa32830)













