## Examine os métodos Details e Delete de um aplicativo ASP.NET Core

Abra o controlador Movie e examine o método Details:

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

O mecanismo de scaffolding MVC que criou este método de ação adiciona um comentário mostrando uma solicitação HTTP que invoca o método.
Nesse caso, é uma solicitação GET com três segmentos de URL, o controlador Movies, o método Details e um valor id.
Lembre-se de que esses segmentos são definidos em Program.cs.

~~~ c #

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

~~~

O EF facilita a pesquisa de dados usando o método FirstOrDefaultAsync. 
Um recurso de segurança importante interno do método é que o código verifica se o método de pesquisa encontrou um filme antes de tentar fazer algo com ele.
Por exemplo, um hacker pode introduzir erros no site alterando a URL criada pelos links de http://localhost:{PORT}/Movies/Details/1 para algo como http://localhost:{PORT}/Movies/Details/12345 (ou algum outro valor que não representa um filme real). 
Se você não marcou um filme nulo, o aplicativo gerará uma exceção.

Examine os métodos Delete e DeleteConfirmed.

~~~ c #
// GET: Movies/Delete/5
public async Task<IActionResult> Delete(int? id)
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

// POST: Movies/Delete/5
[HttpPost, ActionName("Delete")]
[ValidateAntiForgeryToken]
public async Task<IActionResult> DeleteConfirmed(int id)
{
    var movie = await _context.Movie.FindAsync(id);
    if (movie != null)
    {
        _context.Movie.Remove(movie);
    }

    await _context.SaveChangesAsync();
    return RedirectToAction(nameof(Index));
}
~~~

Observe que o método HTTP GET Delete não exclui o filme especificado, mas retorna uma exibição do filme em que você pode enviar (HttpPost) a exclusão. 
A execução de uma operação de exclusão em resposta a uma solicitação GET (ou, de fato, a execução de uma operação de edição, criação ou qualquer outra operação que altera dados) abre uma falha de segurança.

O método [HttpPost] que exclui os dados é chamado DeleteConfirmed para fornecer ao método HTTP POST um nome ou uma assinatura exclusiva.

As duas assinaturas de método são mostradas abaixo:

~~~ c #
// GET: Movies/Delete/5
public async Task<IActionResult> Delete(int? id)
{
~~~

~~~ c #
// POST: Movies/Delete/5
[HttpPost, ActionName("Delete")]
[ValidateAntiForgeryToken]
public async Task<IActionResult> DeleteConfirmed(int id)
{
~~~

O CLR (Common Language Runtime) exige que os métodos sobrecarregados tenham uma assinatura de parâmetro exclusiva (mesmo nome de método, mas uma lista diferente de parâmetros). 
No entanto, aqui você precisa de dois métodos Delete – um para GET e outro para POST – que têm a mesma assinatura de parâmetro. (Ambos precisam aceitar um único inteiro como parâmetro.)

Há duas abordagens para esse problema e uma delas é fornecer aos métodos nomes diferentes. Foi isso o que o mecanismo de scaffolding fez no exemplo anterior. 
No entanto, isso apresenta um pequeno problema: o ASP.NET mapeia os segmentos de URL para os métodos de ação por nome e se você renomear um método, o roteamento normalmente não conseguirá encontrar esse método. 
A solução é o que você vê no exemplo, que é adicionar o atributo ActionName("Delete") ao método DeleteConfirmed.
Esse atributo executa o mapeamento para o sistema de roteamento, de modo que uma URL que inclui /Delete/ para uma solicitação POST encontre o método DeleteConfirmed.

Outra solução alternativa comum para métodos que têm nomes e assinaturas idênticos é alterar artificialmente a assinatura do método POST para incluir um parâmetro extra (não utilizado). 
Foi isso o que fizemos em uma postagem anterior quando adicionamos o parâmetro notUsed. 

Você pode fazer o mesmo aqui para o método [HttpPost] Delete:

~~~ c #
// POST: Movies/Delete/6
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Delete(int id, bool notUsed)
~~~


