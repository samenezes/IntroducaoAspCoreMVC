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

~~~ C #

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

Quando o sistema de scaffolding criou a exibição de Edição, ele examinou a classe Movie e o código criado para renderizar os elementos <label> e <input> de cada propriedade da classe. 
O seguinte exemplo mostra a exibição de Edição que foi gerada pelo sistema de scaffolding do Visual Studio:

~~~ CSHTML #

@model MvcMovie.Models.Movie

@{
    ViewData["Title"] = "Edit";
}

<h1>Edit</h1>

<h4>Movie</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="Edit">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input type="hidden" asp-for="Id" />
            <div class="form-group">
                <label asp-for="Title" class="control-label"></label>
                <input asp-for="Title" class="form-control" />
                <span asp-validation-for="Title" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="ReleaseDate" class="control-label"></label>
                <input asp-for="ReleaseDate" class="form-control" />
                <span asp-validation-for="ReleaseDate" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Genre" class="control-label"></label>
                <input asp-for="Genre" class="form-control" />
                <span asp-validation-for="Genre" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Price" class="control-label"></label>
                <input asp-for="Price" class="form-control" />
                <span asp-validation-for="Price" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Save" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-action="Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}

~~~

Observe como o modelo de exibição tem uma instrução @model MvcMovie.Models.Movie na parte superior do arquivo. 
@model MvcMovie.Models.Movie especifica que a exibição espera que o modelo de exibição seja do tipo Movie.

O código com scaffolding usa vários métodos de Auxiliares de Marcação para simplificar a marcação HTML. 
O Auxiliar de Marcação de Rótulo exibe o nome do campo (“Title”, “ReleaseDate”, “Genre” ou “Price”). 
O Auxiliar de Marcação de Entrada renderiza um elemento <input> HTML. 
O Auxiliar de Marcação de Validação exibe todas as mensagens de validação associadas a essa propriedade.

Execute o aplicativo e navegue para a URL /Movies. Clique em um link Editar. No navegador, exiba a origem da página. 
O HTML gerado para o elemento <form> é mostrado abaixo.

~~~ HTML #

<form action="/Movies/Edit/7" method="post">
    <div class="form-horizontal">
        <h4>Movie</h4>
        <hr />
        <div class="text-danger" />
        <input type="hidden" data-val="true" data-val-required="The ID field is required." id="ID" name="ID" value="7" />
        <div class="form-group">
            <label class="control-label col-md-2" for="Genre" />
            <div class="col-md-10">
                <input class="form-control" type="text" id="Genre" name="Genre" value="Western" />
                <span class="text-danger field-validation-valid" data-valmsg-for="Genre" data-valmsg-replace="true"></span>
            </div>
        </div>
        <div class="form-group">
            <label class="control-label col-md-2" for="Price" />
            <div class="col-md-10">
                <input class="form-control" type="text" data-val="true" data-val-number="The field Price must be a number." data-val-required="The Price field is required." id="Price" name="Price" value="3.99" />
                <span class="text-danger field-validation-valid" data-valmsg-for="Price" data-valmsg-replace="true"></span>
            </div>
        </div>
        <!-- Markup removed for brevity -->
        <div class="form-group">
            <div class="col-md-offset-2 col-md-10">
                <input type="submit" value="Save" class="btn btn-default" />
            </div>
        </div>
    </div>
    <input name="__RequestVerificationToken" type="hidden" value="CfDJ8Inyxgp63fRFqUePGvuI5jGZsloJu1L7X9le1gy7NCIlSduCRx9jDQClrV9pOTTmqUyXnJBXhmrjcUVDJyDUMm7-MF_9rK8aAZdRdlOri7FmKVkRe_2v5LIHGKFcTjPrWPYnc9AdSbomkiOSaTEg7RU" />
</form>

~~~


Os elementos <input> estão em um elemento HTML <form> cujo atributo action está definido para ser postado para a URL /Movies/Edit/id.
Os dados de formulário serão postados com o servidor quando o botão Save receber um clique. 
A última linha antes do elemento </form> de fechamento mostra o token XSRF oculto gerado pelo Auxiliar de Marcação de Formulário.

## Processando a solicitação POST

A lista a seguir mostra a versão [HttpPost] do método de ação Edit.

~~~ C #
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
O atributo [ValidateAntiForgeryToken] valida o token XSRF oculto gerado pelo gerador de tokens antifalsificação no Auxiliar de Marcação de Formulário

O sistema de model binding usa os valores de formulário postados e cria um objeto Movie que é passado como o parâmetro movie. 
A propriedade ModelState.IsValid verifica se os dados enviados no formulário podem ser usados para modificar (editar ou atualizar) um objeto Movie. 
Se os dados forem válidos, eles serão salvos. 
Os dados de filmes atualizados (editados) são salvos no banco de dados chamando o método SaveChangesAsync do contexto de banco de dados. 
Depois de salvar os dados, o código redireciona o usuário para o método de ação Index da classe MoviesController, que exibe a coleção de filmes, incluindo as alterações feitas recentemente.

Antes que o formulário seja postado no servidor, a validação do lado do cliente verifica as regras de validação nos campos. 
Se houver erros de validação, será exibida uma mensagem de erro e o formulário não será postado. 
Se o JavaScript estiver desabilitado, você não terá a validação do lado do cliente, mas o servidor detectará os valores postados que 
não forem válidos e os valores de formulário serão exibidos novamente com mensagens de erro. 
Mais adiante no tutorial, examinamos a Validação de Modelos mais detalhadamente. 
O Auxiliar de Marcação de Validação no modelo de exibição Views/Movies/Edit.cshtml é responsável por exibir as mensagens de erro apropriadas.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/55158a41-09e5-4d48-b0dd-e6f7fd2b3a12)

Todos os métodos HttpGet no controlador de filme seguem um padrão semelhante.
Eles obtêm um objeto de filme (ou uma lista de objetos, no caso de Index) e passam o objeto (modelo) para a exibição.
O método Create passa um objeto de filme vazio para a exibição Create. 
Todos os métodos que criam, editam, excluem ou, de outro modo, modificam dados fazem isso na sobrecarga [HttpPost] do método.
A modificação de dados em um método HTTP GET é um risco de segurança. 
A modificação de dados em um método HTTP GET também viola as melhores práticas de HTTP e o padrão REST de arquitetura, que especifica que as solicitações GET não devem alterar o estado do aplicativo. 
Em outras palavras, a execução de uma operação GET deve ser uma operação segura que não tem efeitos colaterais e não modifica os dados persistentes.







