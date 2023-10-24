# Adicionar uma exibição a um aplicativo ASP.NET Core MVC

## Agora você modificará a classe HelloWorldController para usar arquivos de exibição.

Atualmente, o método Index retorna uma cadeia de caracteres com uma mensagem na classe do controlador. Na classe HelloWorldController, substitua o método Index pelo seguinte código:
~~~c#
public IActionResult Index()
{
    return View();
}
~~~

O código anterior:

Chama o método View do controlador.
Usa um modelo de exibição para gerar uma resposta HTML.
Métodos do controlador:

São chamados demétodos de ação. Por exemplo, o método de ação Index no código anterior.
Geralmente, retorna IActionResult ou uma classe ou derivada de ActionResult, não um tipo como string.

##Adicionar uma exibição

Clique com o botão direito do mouse na pasta Exibições e, em seguida, Adicionar >Nova Pasta e nomeie a pasta HelloWorld.

Clique com o botão direito do mouse na pasta Views/HelloWorld e, em seguida, clique em Adicionar > Novo Item.

Na caixa de diálogo Adicionar Novo Item – MvcMovie:

Na caixa de pesquisa no canto superior direito, insira exibição
Selecione Razor Exibição - Vazio
Mantenha o valor da caixa Nome, Index.cshtml.
Selecione Adicionar

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/084da3f3-2e4f-42f8-b2f6-fc9a9cff4f67)

Substitua o conteúdo do arquivo de exibição Views/HelloWorld/Index.cshtmlRazor pelo seguinte:

~~~CSHTML#
@{
    ViewData["Title"] = "Index";
}

<h2>Index</h2>

<p>Hello from our View Template!</p>
 ~~~

Acesse o diretório https://localhost:{PORT}/HelloWorld:

O método Index no HelloWorldController executou a instrução return View();, que especificou que o método deve usar um arquivo de modelo de exibição para renderizar uma resposta para o navegador.

Como o nome do arquivo de modo de exibição não foi especificado, o MVC é padronizado para usar o arquivo de exibição padrão. Quando o nome do arquivo de exibição não é especificado, a exibição padrão é retornada. A exibição padrão tem o mesmo nome que o método de ação, Index neste exemplo. O modelo de exibição /Views/HelloWorld/Index.cshtml é usado.

A imagem a seguir mostra a cadeia de caracteres "Olá aqui do nosso Modelo de Exibição!" nativa do código do modo de exibição:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/70d4a4f1-ea2e-4692-8c08-b4e73eef6db3)

##Alterar exibições e páginas de layout

Selecione os links de menu MvcMovie, Home e Privacy. Cada página mostra o mesmo layout de menu. O layout do menu é implementado no arquivo Views/Shared/_Layout.cshtml.

Abra o arquivo Views/Shared/_Layout.cshtml .

Os modelos de Layout permitem:

Especificar o layout do contêiner HTML de um site em um só lugar.
Aplicar o layout do contêiner HTML em várias páginas no site.
Localize a linha @RenderBody(). RenderBody é um espaço reservado em que todas as páginas específicas à exibição criadas são mostradas, encapsuladas na página de layout. Por exemplo, se você selecionar o link Privacy, a exibição Views/Home/Privacy.cshtml será renderizada dentro do método RenderBody.

##Alterar o título, o rodapé e o link de menu no arquivo de layout

Substitua o conteúdo do arquivo Views/Shared/_Layout.cshtml pela seguinte marcação. 

As alterações são realçadas:

~~~CSHTML#
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Movie App</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container-fluid">
                <a class="navbar-brand" asp-area="" asp-controller="Movies" asp-action="Index">Movie App</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2022 - Movie App - <a asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
        </div>
    </footer>
    <script src="~/lib/jquery/dist/jquery.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
 ~~~
A marcação anterior fez as seguintes alterações:

Três ocorrências de MvcMovie para Movie App.
O elemento de âncora <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">MvcMovie</a> para <a class="navbar-brand" asp-controller="Movies" asp-action="Index">Movie App</a>.
Na marcação anterior, o valor do atributo e o asp-area=""atributo do Auxiliar de Marca de Âncora e o valor do atributo foram omitidos porque esse aplicativo não está usando Áreas.

Observação: O controlador Movies não foi implementado. Nesse ponto, o link Movie App não está funcionando.

Salve as alterações e selecione o link Privacy. Observe como o título na guia do navegador agora exibe PrivacyPolítica - Aplicativo de Filme, em vez de PrivacyPolítica - MvcMovie

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/3d1ac8b3-ac98-4afe-ab52-f504321a6c8a)

Selecione o link Home.

Observe que o título e o texto de âncora mostram Aplicativo de Filme. As alterações foram feitas uma vez no modelo de layout e que todas as páginas do site refletem o novo texto do link e o novo título.

Examine o arquivo Views/_ViewStart.cshtml:

~~~CSHTML#
@{
    Layout = "_Layout";
}
~~~
O arquivo Views/_ViewStart.cshtml traz o arquivo Views/Shared/_Layout.cshtml para cada exibição. A propriedade Layout pode ser usada para definir outra exibição de layout ou defina-a como null para que nenhum arquivo de layout seja usado.

Abra o arquivo de exibição Views/HelloWorld/Index.cshtml.

Altere o título e o elemento h2 conforme realçado a seguir:

~~~CSHTML#
@{
    ViewData["Title"] = "Movie List";
}

<h2>My Movie List</h2>

<p>Hello from our View Template!</p>
~~~
O título e o elemento h2 são ligeiramente diferentes para que fique claro qual parte do código altera a exibição.

ViewData["Title"] = "Movie List"; no código acima define a propriedade Title do dicionário ViewData como “Lista de Filmes”. A propriedade Title é usada no elemento HTML <title> na página de layout:

~~~CSHTML#
<title>@ViewData["Title"] - Movie App</title>
~~~

Salve as alterações e navegue para https://localhost:{PORT}/HelloWorld.

Observe que as seguintes alterações foram feitas:

Título do navegador.
Título primário.
Títulos secundários.
Se não houver alterações no navegador, pode ser que o conteúdo que está sendo exibido esteja armazenado em cache. Pressione Ctrl+F5 no navegador para forçar a resposta do servidor a ser carregada. O título do navegador é criado com ViewData["Title"] que definimos no modelo de exibição Index.cshtml e o “- Aplicativo de Filme” adicionado no arquivo de layout.

O conteúdo do modelo de exibição Index.cshtml é mesclado com o modelo de exibição Views/Shared/_Layout.cshtml . Uma única resposta HTML é enviada ao navegador. Os modelos de layout facilitam a realização de alterações que se aplicam a todas as páginas de um aplicativo.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/509ff42b-6ae0-4e31-a177-95c4eaa1413b)

No entanto, a pequena quantidade de "dados", a mensagem "Olá aqui do nosso Modelo de Exibição!", é nativa do código.
O aplicativo MVC tem um “V” (exibição), um “C” (controlador), mas ainda nenhum “M” (modelo).


##Passando dados do controlador para a exibição

As ações do controlador são invocadas em resposta a uma solicitação de URL de entrada. Uma classe de controlador é o local em que o código é escrito e que manipula as solicitações recebidas do navegador. O controlador recupera dados de uma fonte de dados e decide qual tipo de resposta será enviada novamente para o navegador. Modelos de exibição podem ser usados em um controlador para gerar e formatar uma resposta HTML para o navegador.

Os controladores são responsáveis por fornecer os dados necessários para que um modelo de exibição renderize uma resposta.

Os modelos de exibição não devem:

Processar lógicas de negócios
Interagir diretamente com algum banco de dados.
O modelo de exibição deve funcionar somente com os dados fornecidos pelo controlador. Manter essa “separação de preocupações” ajuda a manter o código:

Limpo.
Testável.
Descomplicado em relação à manutenção.
Atualmente, o método Welcome na classe HelloWorldController usa um name e um parâmetro IDe, em seguida, gera os valores de saída diretamente no navegador.

Em vez de fazer com que o controlador renderize a resposta como uma cadeia de caracteres, altere o controlador para que ele use um modelo de exibição. O modelo de exibição gera uma resposta dinâmica, o que significa que é necessário passar os dados apropriados do controlador para a exibição para gerar a resposta. Faça isso fazendo com que o controlador coloque os dados dinâmicos (parâmetros) que o modelo de exibição precisa em um dicionário ViewData. O modelo de exibição pode então acessar os dados dinâmicos.

Em HelloWorldController.cs, altere o método Welcome para adicionar um valor Message e NumTimes ao dicionário ViewData.

O dicionário ViewData é um objeto dinâmico, o que significa que qualquer tipo pode ser usado. O objeto ViewData não tem propriedades definidas até que algo seja adicionado. O sistema de model binding do MVC mapeia automaticamente os parâmetros nomeados name e numTimes da cadeia de caracteres de consulta para os parâmetros do método. O HelloWorldController completo:



