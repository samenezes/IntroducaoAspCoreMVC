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






