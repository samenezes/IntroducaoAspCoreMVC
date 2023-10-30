# Adicionar a validação a um aplicativo ASP.NET Core MVC

1. A lógica de validação é adicionada ao modelo Movie.
2. Você garante que as regras de validação sejam impostas sempre que um usuário criar ou editar um filme.

   
# Mantendo o processo DRY

Um dos princípios de design do MVC é o DRY (“Don't Repeat Yourself”).
O ASP.NET Core MVC incentiva você a especificar a funcionalidade ou o comportamento somente uma vez e, em seguida, refleti-lo em qualquer lugar de um aplicativo. 
Isso reduz a quantidade de código que você precisa escrever e faz com que o código escrito seja menos propenso a erros e mais fácil de testar e manter.

O suporte de validação fornecido pelo MVC e pelo Entity Framework Core Code First é um bom exemplo do princípio DRY em ação. 
Especifique as regras de validação de forma declarativa em um único lugar (na classe de modelo) e as regras são impostas em qualquer lugar no aplicativo.

Adicionar regras de validação ao modelo de filme
O namespace DataAnnotations fornece um conjunto de atributos de validação internos aplicados de forma declarativa a uma classe ou propriedade. 
DataAnnotations também contém atributos de formatação como DataType, que ajudam com a formatação e não fornecem validação.

Atualize a classe Movie para aproveitar os atributos de validação internos Required, StringLength, RegularExpression e Range o atributo de formatação DataType.

~~~ c #
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }

    [StringLength(60, MinimumLength = 3)]
    [Required]
    public string? Title { get; set; }

    [Display(Name = "Release Date")]
    [DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }

    [Range(1, 100)]
    [DataType(DataType.Currency)]
    [Column(TypeName = "decimal(18, 2)")]
    public decimal Price { get; set; }    

    [RegularExpression(@"^[A-Z]+[a-zA-Z\s]*$")]
    [Required]
    [StringLength(30)]
    public string? Genre { get; set; }
    
    [RegularExpression(@"^[A-Z]+[a-zA-Z0-9""'\s-]*$")]
    [StringLength(5)]
    [Required]
    public string? Rating { get; set; }
}

~~~


Os atributos de validação especificam o comportamento que você deseja impor nas propriedades de modelo às quais eles são aplicados:

 1. Os atributos Required e MinimumLength indicam que uma propriedade deve ter um valor; porém, nada impede que um usuário insira um espaço em branco para atender a essa validação.
    
 2. O atributo RegularExpression é usado para limitar quais caracteres podem ser inseridos. No código anterior, "Gênero": 
        Deve usar apenas letras.
        A primeira letra deve ser maiúscula. Espaços em branco são permitidos, enquanto números e caracteres especiais não são.

 3. A "Classificação" RegularExpression:
    Exige que o primeiro caractere seja uma letra maiúscula.
    Permite caracteres especiais e números nos espaços subsequentes. "PG-13" é válido para uma classificação, mas é um erro em "Gênero".
    
 4. O atributo Range restringe um valor a um intervalo especificado.
 5. O atributo StringLength permite definir o tamanho máximo de uma propriedade de cadeia de caracteres e, opcionalmente, seu tamanho mínimo.
 6. Os tipos de valor (como decimal, int, float, DateTime) são inerentemente necessários e não precisam do atributo [Required].

Ter as regras de validação automaticamente impostas pelo ASP.NET Core ajuda a tornar seu aplicativo mais robusto. 
Também garante que você não se esqueça de validar algo e inadvertidamente permita dados incorretos no banco de dados.

## Interface do usuário de erro de validação

Execute o aplicativo e navegue para o controlador Movies.

Selecione o link Criar Novo para adicionar um novo filme.
Preencha o formulário com alguns valores inválidos. Assim que a validação do lado do cliente do jQuery detecta o erro, ela exibe uma mensagem de erro.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/885568ec-c51b-481e-b004-862f2f46a48e)

Observe como o formulário renderizou automaticamente uma mensagem de erro de validação apropriada em cada campo que contém um valor inválido. 
Os erros são impostos no lado do cliente (usando o JavaScript e o jQuery) e no lado do servidor (caso um usuário tenha o JavaScript desabilitado).

Uma vantagem significativa é que você não precisa alterar uma única linha de código na classe MoviesController ou na exibição Create.cshtml para habilitar essa interface do usuário de validação. 
O controlador e as exibições criados anteriormente neste tutorial selecionaram automaticamente as regras de validação especificadas com atributos de validação nas propriedades da classe de modelo Movie. 
Teste a validação usando o método de ação Edit e a mesma validação é aplicada.

Os dados de formulário não serão enviados para o servidor enquanto houver erros de validação do lado do cliente.

## Como funciona a validação
Talvez você esteja se perguntando como a interface do usuário de validação foi gerada sem atualizações do código no controlador ou nas exibições.
O código a seguir mostra os dois métodos Create.

~~~ c #
// GET: Movies/Create
public IActionResult Create()
{
    return View();
}

// POST: Movies/Create
// To protect from overposting attacks, enable the specific properties you want to bind to.
// For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create([Bind("Id,Title,ReleaseDate,Genre,Price,Rating")] Movie movie)
{
    if (ModelState.IsValid)
    {
        _context.Add(movie);
        await _context.SaveChangesAsync();
        return RedirectToAction(nameof(Index));
    }
    return View(movie);
}
~~~

O primeiro método de ação (HTTP GET) Create exibe o formulário Criar inicial. A segunda versão ([HttpPost]) manipula a postagem de formulário.
O segundo método Create (a versão [HttpPost]) chama ModelState.IsValid para verificar se o filme tem erros de validação.
A chamada a esse método avalia os atributos de validação que foram aplicados ao objeto. Se o objeto tiver erros de validação, o método Create exibirá o formulário novamente.
Se não houver erros, o método salvará o novo filme no banco de dados. 
Em nosso exemplo de filme, o formulário não é postado no servidor quando há erros de validação detectados no lado do cliente; 
o segundo método Create nunca é chamado quando há erros de validação do lado do cliente. 
Se você desabilitar o JavaScript no navegador, a validação do cliente será desabilitada e você poderá testar o método Create HTTP POST ModelState.IsValid detectando erros de validação.
Defina um ponto de interrupção no método [HttpPost] Create e verifique se o método nunca é chamado; a validação do lado do cliente não enviará os dados de formulário quando forem detectados erros de validação. 
Se você desabilitar o JavaScript no navegador e, em seguida, enviar o formulário com erros, o ponto de interrupção será atingido. Você ainda pode obter uma validação completa sem o JavaScript.


A imagem a seguir mostra como desabilitar o JavaScript no navegador Firefox.
![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/c14aaa06-3bc3-417e-a107-3cc0ddacc9ed)


A imagem a seguir mostra como desabilitar o JavaScript no navegador Chrome.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/71a80ccc-ac2f-4850-8d9c-0a0d3d935fe0)


Depois de desabilitar o JavaScript, poste os dados inválidos e execute o depurador em etapas.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/e91bb6e7-ebfc-4826-ac91-20c6814a6b33)

A parte do modo de exibição Create.cshtml é mostrada na seguinte marcação:

~~~ html #

<h4>Movie</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form asp-action="Create">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Title" class="control-label"></label>
                <input asp-for="Title" class="form-control" />
                <span asp-validation-for="Title" class="text-danger"></span>
            </div>

            @*Markup removed for brevity.*@

~~~

a marcação anterior é usada pelos métodos de ação para exibir o formulário inicial e exibi-lo novamente em caso de erro.

O Auxiliar de Marcação de Entrada usa os atributos de DataAnnotations e produz os atributos HTML necessários para a Validação do jQuery no lado do cliente. 
O Auxiliar de Marcação de Validação exibe erros de validação. 

O que é realmente interessante nessa abordagem é que o controlador nem o modelo de exibição Create sabem nada sobre as regras de validação reais que estão sendo impostas ou as mensagens de erro específicas exibidas.
As regras de validação e as cadeias de caracteres de erro são especificadas somente na classe Movie. 
Essas mesmas regras de validação são aplicadas automaticamente à exibição Edit e a outros modelos de exibição que podem ser criados e que editam o modelo.

Quando você precisar alterar a lógica de validação, faça isso exatamente em um lugar, adicionando atributos de validação ao modelo (neste exemplo, a classe Movie). 
Você não precisa se preocupar se diferentes partes do aplicativo estão inconsistentes com a forma como as regras são impostas – toda a lógica de validação será definida em um lugar e usada em todos os lugares. 
Isso mantém o código muito limpo e torna-o mais fácil de manter e desenvolver.
Além disso, isso significa que você respeitará totalmente o princípio DRY.


## Usando atributos DataType

Abra o arquivo Movie.cs e examine a classe Movie. 
O namespace System.ComponentModel.DataAnnotations fornece atributos de formatação, além do conjunto interno de atributos de validação. 
Já aplicamos um valor de enumeração DataType à data de lançamento e aos campos de preço. 
O código a seguir mostra as propriedades ReleaseDate e Price com o atributo DataType apropriado.

~~~ c #

[Display(Name = "Release Date")]
[DataType(DataType.Date)]              //essa linha
public DateTime ReleaseDate { get; set; }

[Range(1, 100)]
[DataType(DataType.Currency)]         //essa linha  
[Column(TypeName = "decimal(18, 2)")]
public decimal Price { get; set; }

~~~


Os atributos DataType fornecem dicas apenas para que o mecanismo de exibição formate os dados e fornece elementos/atributos como <a> para as URLs e <a href="mailto:EmailAddress.com"> para o email.
Use o atributo RegularExpression para validar o formato dos dados. 
O atributo DataType é usado para especificar um tipo de dados mais específico do que o tipo intrínseco de banco de dados; eles não são atributos de validação.
Nesse caso, apenas desejamos acompanhar a data, não a hora.
A Enumeração DataType fornece muitos tipos de dados, como Date, Time, PhoneNumber, Currency, EmailAddress e muito mais. 
O atributo DataType também pode permitir que o aplicativo forneça automaticamente recursos específicos a um tipo. 
Por exemplo, um link mailto: pode ser criado para DataType.EmailAddress e um seletor de data pode ser fornecido para DataType.Date em navegadores que dão suporte a HTML5. 
Os atributos DataType emitem atributos data- HTML 5 (ou "data dash") que são reconhecidos pelos navegadores HTML 5.
Os atributos DataType não fornecem nenhuma validação.

DataType.Date não especifica o formato da data exibida. Por padrão, o campo de dados é exibido de acordo com os formatos padrão com base nas CultureInfo do servidor.

O atributo DisplayFormat é usado para especificar explicitamente o formato de data:

~~~ c #
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
public DateTime ReleaseDate { get; set; }
~~~

A configuração ApplyFormatInEditMode especifica que a formatação também deve ser aplicada quando o valor é exibido em uma caixa de texto para edição.
(Talvez você não deseje isso em alguns campos – por exemplo, para valores de moeda, provavelmente, você não deseja exibir o símbolo de moeda na caixa de texto para edição.)

Você pode usar o atributo DisplayFormat por si só, mas geralmente é uma boa ideia usar o atributo DataType.
O atributo DataType transmite a semântica dos dados, ao invés de apresentar como renderizá-lo em uma tela e oferece os seguintes benefícios que você não obtém com DisplayFormat:

O navegador pode habilitar os recursos do HTML5 (por exemplo, mostrar um controle de calendário, o símbolo de moeda apropriado à localidade, links de email, etc.)

Por padrão, o navegador renderizará os dados usando o formato correto de acordo com a localidade.

O atributo DataType pode permitir que o MVC escolha o modelo de campo correto para renderizar os dados (o DisplayFormat, se usado por si só, usa o modelo de cadeia de caracteres).


A validação do jQuery não funciona com os atributos Range e DateTime. Por exemplo, o seguinte código sempre exibirá um erro de validação do lado do cliente, mesmo quando a data estiver no intervalo especificado:

[Range(typeof(DateTime), "1/1/1966", "1/1/2020")]

Você precisará desabilitar a validação de data do jQuery para usar o atributo Range com DateTime. Geralmente, não é uma boa prática compilar datas rígidas nos modelos e, portanto, o uso do atributo Range e de DateTime não é recomendado.

O seguinte código mostra como combinar atributos em uma linha:

~~~ c #

using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace MvcMovie.Models;

public class Movie
{
    public int Id { get; set; }
    [StringLength(60, MinimumLength = 3)]
    public string Title { get; set; }
    [Display(Name = "Release Date"), DataType(DataType.Date)]
    public DateTime ReleaseDate { get; set; }
    [RegularExpression(@"^[A-Z]+[a-zA-Z\s]*$"), Required, StringLength(30)]
    public string Genre { get; set; }
    [Range(1, 100), DataType(DataType.Currency)]
    [Column(TypeName = "decimal(18, 2)")]
    public decimal Price { get; set; }
    [RegularExpression(@"^[A-Z]+[a-zA-Z0-9""'\s-]*$"), StringLength(5)]
    public string Rating { get; set; }
}

~~~



