# Adicionar um novo campo a um aplicativo ASP.NET Core MVC

As Migrações do Entity Framework Code First são usadas para:

 1. Adicionar um novo campo ao modelo.
 2. Migrar o novo campo para o banco de dados.

# Ao usar o Code First do EF para criar automaticamente um banco de dados, o Code First:

 1. Adiciona uma tabela ao banco de dados para rastrear o esquema do banco de dados.
 2. Verifica se o banco de dados está em sincronia com as classes de modelo das quais foi gerado. Se ele não estiver sincronizado, o EF gerará uma exceção.
    Isso facilita a descoberta de problemas de código/banco de dados inconsistente.
    

## Adicionar uma Propriedade de Classificação ao Modelo de Filme

Adiciona uma propriedade Rating a Models/Movie.cs:

~~~ c #

using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }
    public string? Title { get; set; }

    [Display(Name = "Release Date")]
    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }
    public string? Genre { get; set; }
    
    [Column(TypeName = "decimal(18, 2)")]
    public decimal Price { get; set; }
    public string? Rating {  get; set; }  // essa linha
}

~~~

## Criar o aplicativo

Ctrl+Shift+B

Como adicionou um novo campo à classe Movie, você também precisará atualizar a lista de permissões de associação para que essa nova propriedade seja incluída. 
Em MoviesController.cs, atualize o atributo [Bind] dos métodos de ação Create e Edit para incluir a propriedade Rating:

~~~ c #
[Bind("Id,Title,ReleaseDate,Genre,Price,Rating")]
~~~

Atualize os modelos de exibição para exibir, criar e editar a nova propriedade Rating na exibição do navegador.

Edite o arquivo /Views/Movies/Index.cshtml e adicione um campo Rating:

~~~ cshtml #

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
            <th>                                                             //essa linha
                @Html.DisplayNameFor(model => model.Movies![0].Rating)       //essa linha
            </th>                                                            //essa linha
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
                <td>                                                //essa linha
                    @Html.DisplayFor(modelItem => item.Rating)      //essa linha 
                </td>                                               //essa linha
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

Atualize o /Views/Movies/Create.cshtml com um campo Rating.

Copie/cole o “grupo de formulário” anterior e permita que o IntelliSense ajude você a atualizar os campos. 
O IntelliSense funciona com os Auxiliares de Marcação.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/cf220cd6-5d5e-4461-91cf-6f23c39019ba)

Atualize os modelos restantes.

Atualize a classe SeedData para que ela forneça um valor para a nova coluna.
Uma alteração de amostra é mostrada abaixo, mas é recomendável fazer essa alteração em cada new Movie.

~~~ c #

new Movie
{
    Title = "When Harry Met Sally",
    ReleaseDate = DateTime.Parse("1989-1-11"),
    Genre = "Romantic Comedy",
    Rating = "R",       //essa linha
    Price = 7.99M
},

~~~


O aplicativo não funcionará até que o BD seja atualizado para incluir o novo campo. Se ele tiver sido executado agora, o seguinte SqlException será gerado:

SqlException: Invalid column name 'Rating'.

Este erro ocorre porque a classe de modelo Movie atualizada é diferente do esquema da tabela Movie do banco de dados existente. (Não há nenhuma coluna Rating na tabela de banco de dados.)

Existem algumas abordagens para resolver o erro:

Faça com que o Entity Framework remova automaticamente e recrie o banco de dados com base no novo esquema de classe de modelo. 
Essa abordagem é muito conveniente no início do ciclo de desenvolvimento, quando você está fazendo o desenvolvimento ativo em um banco de dados de teste. 
Ela permite que você desenvolva rapidamente o modelo e o esquema de banco de dados juntos.
No entanto, a desvantagem é que você perde os dados existentes no banco de dados – portanto, você não deseja usar essa abordagem em um banco de dados de produção! 
Muitas vezes, o uso de um inicializador para propagar um banco de dados com os dados de teste automaticamente é uma maneira produtiva de desenvolver um aplicativo.
Isso é uma boa abordagem para o desenvolvimento inicial e ao usar o SQLite.

Modifique explicitamente o esquema do banco de dados existente para que ele corresponda às classes de modelo. 
A vantagem dessa abordagem é que você mantém os dados. Faça essa alteração manualmente ou criando um script de alteração de banco de dados.

Use as Migrações do Code First para atualizar o esquema de banco de dados.

Para este tutorial, as Migrações do Code First são usadas.

No menu Ferramentas, selecione Gerenciador de Pacotes NuGet > Console do Gerenciador de Pacotes.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/bb32ab69-48c6-4270-b762-f260c51822ef)

No PMC, insira os seguintes comandos:

~~~ powershell #

Add-Migration Rating
Update-Database

~~~
O comando Add-Migration informa a estrutura de migração para examinar o atual modelo Movie com o atual esquema de BD Movie e criar o código necessário para migrar o BD para o novo modelo.

O nome “Classificação” é arbitrário e é usado para nomear o arquivo de migração.
É útil usar um nome significativo para o arquivo de migração.

Se você excluir todos os registros do BD, o método de inicialização propagará o BD e incluirá o campo Rating.

Execute o aplicativo e verifique se você pode criar, editar e exibir filmes com um campo Rating.
