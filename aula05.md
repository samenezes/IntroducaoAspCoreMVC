# Trabalhar com um banco de dados em um aplicativo MVC ASP.NET Core

O objeto MvcMovieContext cuida da tarefa de se conectar ao banco de dados e mapear objetos Movie para registros do banco de dados.
O contexto do banco de dados é registrado com o contêiner Injeção de dependência no arquivo

~~~ c #

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<MvcMovieContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("MvcMovieContext")));

~~~

O sistema de Configuração do ASP.NET Core lê a chave ConnectionString. 
Para o desenvolvimento local, ele obtém a cadeia de conexão do arquivo appsettings.json:

~~~ json #
"ConnectionStrings": {
  "MvcMovieContext": "Data Source=MvcMovieContext-ea7a4069-f366-4742-bd1c-3f753a804ce1.db"
}
~~~

Quando o aplicativo é implantado em um servidor de teste ou de produção, uma variável de ambiente pode ser usada para definir a cadeia de conexão como um SQL Server de produção.

SQL Server Express LocalDB
LocalDB:

É uma versão leve do Mecanismo de Banco de Dados do SQL Server Express, instalada por padrão com o Visual Studio.
Inicia sob demanda usando uma cadeia de conexão.
É direcionado para o desenvolvimento de programas. É executado no modo de usuário e, portanto, não há nenhuma configuração complexa.
Por padrão, cria arquivos .mdf no diretório C:/Users/{user}.
Examinar o banco de dados
No menu Exibir, abra SSOX (Pesquisador de Objetos do SQL Server).

Clique com o botão direito do mouse na Movie tabela (dbo.Movie) > Designer de Exibição


![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/4dbc4e56-c6a5-4e06-b0d0-ebd859067d18)


![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/7344e1ae-83bb-4804-abbb-765c141e2e67)

Observe o ícone de chave ao lado de ID. Por padrão, o EF tornará uma propriedade chamada ID a chave primária.

Clique com o botão direito na tabela Movie>Dados de Exibição

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/36c9467c-6368-499a-8400-a9c69744dd7f)

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/4e229624-6a5a-416c-ac59-235ac85b6226)

## Propagar o banco de dados

Crie uma nova classe chamada SeedData na pasta Models. Substitua o código gerado pelo seguinte:

~~~ c #

using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using MvcMovie.Data;
using System;
using System.Linq;

namespace MvcMovie.Models;

public static class SeedData
{
    public static void Initialize(IServiceProvider serviceProvider)
    {
        using (var context = new MvcMovieContext(
            serviceProvider.GetRequiredService<
                DbContextOptions<MvcMovieContext>>()))
        {
            // Look for any movies.
            if (context.Movie.Any())
            {
                return;   // DB has been seeded
            }
            context.Movie.AddRange(
                new Movie
                {
                    Title = "When Harry Met Sally",
                    ReleaseDate = DateTime.Parse("1989-2-12"),
                    Genre = "Romantic Comedy",
                    Price = 7.99M
                },
                new Movie
                {
                    Title = "Ghostbusters ",
                    ReleaseDate = DateTime.Parse("1984-3-13"),
                    Genre = "Comedy",
                    Price = 8.99M
                },
                new Movie
                {
                    Title = "Ghostbusters 2",
                    ReleaseDate = DateTime.Parse("1986-2-23"),
                    Genre = "Comedy",
                    Price = 9.99M
                },
                new Movie
                {
                    Title = "Rio Bravo",
                    ReleaseDate = DateTime.Parse("1959-4-15"),
                    Genre = "Western",
                    Price = 3.99M
                }
            );
            context.SaveChanges();
        }
    }
}

~~~

Se houver um filme no banco de dados, o inicializador de semeadura será retornado e nenhum filme será adicionado.

~~~ c #

if (context.Movie.Any())
{
    return;  // DB has been seeded.
}

~~~

<a name=snippet_"si">

## Adicionar o inicializador de semeadura

Substitua o conteúdo de Program.cs pelo seguinte código. O novo código está realçado.

~~~ c #

using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using MvcMovie.Data;
using MvcMovie.Models; //essa linha 

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<MvcMovieContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("MvcMovieContext")));

// Add services to the container.
builder.Services.AddControllersWithViews();

var app = builder.Build();

using (var scope = app.Services.CreateScope()) //essa linha
{
    var services = scope.ServiceProvider;

    SeedData.Initialize(services);
}                                              //até essa linha

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();

~~~

Exclua todos os registros no banco de dados. Faça isso com os links Excluir no navegador ou no SSOX.

Testar o aplicativo. Force o aplicativo a ser inicializado, chamando o código no arquivo Program.cs) para que o método de propagação seja executado. 
Para forçar a inicialização, feche a janela do prompt de comando que o Visual Studio abriu e reinicie pressionando Ctrl+F5.

O aplicativo mostra os dados propagados.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/b3e5e4e2-ee8c-4f3f-a897-6ace5c27e2c1)








