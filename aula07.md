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

