# Os métodos e as exibições do controlador no ASP.NET Core

Temos um bom começo para o aplicativo de filme, mas a apresentação não é ideal.
Por exemplo, ReleaseDate deveria ser separado em duas palavras.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/7e73539e-f3ce-4df4-8799-369f358d7a1a)

Abra o arquivo Models/Movie.cs e adicione as linhas realçadas mostradas abaixo:

~~~ c #
using System;
using System.ComponentModel.DataAnnotations;         //essa linha
using System.ComponentModel.DataAnnotations.Schema;  // essa linha

namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }
    public string? Title { get; set; }
    
    [Display(Name = "Release Date")]                 //essa linha
    [DataType(DataType.Date)]                        //essa linha
    public DateTime ReleaseDate { get; set; }
    public string? Genre { get; set; }
    [Column(TypeName = "decimal(18, 2)")]           //essa linha
    public decimal Price { get; set; }
}

~~~

DataAnnotations são explicados no próximo tutorial. O atributo Display especifica o que deve ser exibido no nome de um campo (neste caso, “Release Date” em vez de “ReleaseDate”). 
O atributo DataType especifica o tipo de dados (Data) e, portanto, as informações de hora armazenadas no campo não são exibidas.

A anotação de dados [Column(TypeName = "decimal(18, 2)")] é necessária para que o Entity Framework Core possa mapear corretamente o Price para a moeda no banco de dados. 
Procure o controlador Movies e mantenha o ponteiro do mouse pressionado sobre um link Editar para ver a URL de destino.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/bdc956fe-0386-45ce-8646-fe2bf3e42344)

Os links Editar, Detalhes e Excluir são gerados pelo Auxiliar de Marcação de Âncora do MVC Core no arquivo Views/Movies/Index.cshtml.

~~~ cshtml #

        <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |           //essa linha
        <a asp-action="Details" asp-route-id="@item.Id">Details</a> |     //essa linha
        <a asp-action="Delete" asp-route-id="@item.Id">Delete</a>          //essa linha
    </td>
</tr>

~~~

Os Auxiliares de Marcação permitem que o código do servidor participe da criação e renderização de elementos HTML em arquivos do Razor. 
No código acima, o AnchorTagHelper gera dinamicamente o valor do atributo href HTML com base na ID de rota e no método de ação do controlador.
Use a opção Exibir Código-fonte em seu navegador favorito ou as ferramentas do desenvolvedor para examinar a marcação gerada. 
Uma parte do HTML gerado é mostrada abaixo:

~~~ html #

<td>
    <a href="/Movies/Edit/4"> Edit </a> |
    <a href="/Movies/Details/4"> Details </a> |
    <a href="/Movies/Delete/4"> Delete </a>
</td>

~~~

Lembre-se do formato do roteamento definido no arquivo Program.cs:

~~~ c #
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
~~~

O ASP.NET Core converte https://localhost:5001/Movies/Edit/4 de uma solicitação no método de ação Edit do controlador Movies com o parâmetro Id igual a 4. 
(Os métodos do controlador também são conhecidos como métodos de ação.)

Os Auxiliares de Marcação são um dos novos recursos mais populares do ASP.NET Core. 

Abra o controlador Movies e examine os dois métodos de ação Edit.
O código a seguir mostra o método HTTP GET Edit, que busca o filme e popula o formato de edição gerado pelo arquivo Edit.cshtmlRazor do Razor.

~~~ c #

// GET: Movies/Edit/5
public async Task<IActionResult> Edit(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    var movie = await _context.Movie.FindAsync(id);
    if (movie == null)
    {
        return NotFound();
    }
    return View(movie);
}
~~~
O código a seguir mostra o método HTTP POST Edit, que processa os valores de filmes postados:

~~~ c #

// POST: Movies/Edit/5
// To protect from overposting attacks, enable the specific properties you want to bind to.
// For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Edit(int id, [Bind("Id,Title,ReleaseDate,Genre,Price,Rating")] Movie movie)
{
    if (id != movie.Id)
    {
        return NotFound();
    }

    if (ModelState.IsValid)
    {
        try
        {
            _context.Update(movie);
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!MovieExists(movie.Id))
            {
                return NotFound();
            }
            else
            {
                throw;
            }
        }
        return RedirectToAction(nameof(Index));
    }
    return View(movie);
}

~~~

O atributo [Bind] é uma maneira de proteger contra o excesso de postagem. 
Você somente deve incluir as propriedades do atributo [Bind] que deseja alterar. 
ViewModels fornece uma abordagem alternativa para prevenir o excesso de postagem.

Observe se o segundo método de ação Edit é precedido pelo atributo [HttpPost].

~~~ C #

// POST: Movies/Edit/5
// To protect from overposting attacks, enable the specific properties you want to bind to.
// For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]  // AQUI
[ValidateAntiForgeryToken]
public async Task<IActionResult> Edit(int id, [Bind("Id,Title,ReleaseDate,Genre,Price,Rating")] Movie movie)
{
    if (id != movie.Id)
    {
        return NotFound();
    }

    if (ModelState.IsValid)
    {
        try
        {
            _context.Update(movie);
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!MovieExists(movie.Id))
            {
                return NotFound();
            }
            else
            {
                throw;
            }
        }
        return RedirectToAction(nameof(Index));
    }
    return View(movie);
}

~~~

O atributo HttpPost especifica que esse método Edit pode ser invocado somente para solicitações POST. 
Você pode aplicar o atributo [HttpGet] ao primeiro método de edição, mas isso não é necessário porque [HttpGet] é o padrão.

O atributo ValidateAntiForgeryToken é usado para prevenir a falsificação de uma solicitação e é associado a um token antifalsificação gerado no arquivo de exibição de edição (Views/Movies/Edit.cshtml).
O arquivo de exibição de edição gera o token antifalsificação com o Auxiliar de Marcação de Formulário.

~~~ CSHTML #

<form asp-action="Edit">

~~~

O Auxiliar de Marcação de Formulário gera um token antifalsificação oculto que deve corresponder ao token antifalsificação gerado [ValidateAntiForgeryToken] no método Edit do controlador Movies. 
O método HttpGet Edit usa o parâmetro ID de filme, pesquisa o filme usando o método FindAsync do Entity Framework e retorna o filme selecionado para a exibição de Edição. 
Se um filme não for encontrado, NotFound (HTTP 404) será retornado.





