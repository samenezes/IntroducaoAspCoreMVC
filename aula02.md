# Aula 02 Adicionar um controlador a um aplicativo ASP.NET Core MVC

## Arquitetura MVC
O padrão de arquitetura MVC (Model-View-Controller) separa um aplicativo em três componentes principais: Model, View e Controller. 
O padrão MVC ajuda a criar aplicativos que são mais testáveis e fáceis de atualizar comparado aos aplicativos monolíticos tradicionais.

## Os aplicativos baseados no MVC contêm:

> Models: classes que representam os dados do aplicativo. As classes de modelo usam a lógica de validação para impor regras de negócio aos dados. Normalmente, os objetos de modelo recuperam e armazenam o estado do modelo em um banco de dados. Neste tutorial, um modelo Movie recupera dados de filmes de um banco de dados, fornece-os para a exibição ou atualiza-os. O dados atualizados são gravados em um banco de dados.
> Views: exibições são os componentes que exibem a interface do usuário do aplicativo. Em geral, essa interface do usuário exibe os dados de modelo.
> Controladores: Classes que:
Manipular solicitações do navegador.
Recuperar dados do modelo.
Chamar modelos de exibição que retornam uma resposta.

Em um aplicativo MVC, a exibição exibe apenas informações. 
O controlador manipula e responde à entrada e interação do usuário. Por exemplo, o controlador manipula os segmentos da URL e os valores de cadeia de consulta e passa esses valores para o modelo.
O modelo pode usar esses valores para consultar o banco de dados. Por exemplo:

https://localhost:5001/Home/Privacy: especifica o controlador Home e a ação Privacy.
https://localhost:5001/Movies/Edit/5: é uma solicitação para editar o filme com ID=5 usando o controlador Movies e a ação Edit, que são detalhados posteriormente no tutorial.

Os dados de rota serão explicados posteriormente no tutorial.

O padrão de arquitetura MVC separa um aplicativo em três grupos de componentes principais: Modelos, Exibições e Componentes. Esse padrão ajuda a obter a separação de interesses: a lógica da interface do usuário pertence à exibição. A lógica de entrada pertence ao controlador. A lógica de negócios pertence ao modelo. Essa separação ajuda a gerenciar a complexidade ao criar um aplicativo, porque permite trabalhar em um aspecto da implementação por vez, sem afetar o código de outro. Por exemplo, você pode trabalhar no código de exibição sem depender do código da lógica de negócios.

O projeto MVC contém pastas para os Controladores e as Exibições.

## Adicionar um controlador

### No Gerenciador de Soluções, clique com o botão direito do mouse em Controladores > Adicionar > Controlador.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/d825179d-344c-4429-8d75-c1573434043a)

Na caixa de diálogo Adicionar Novo Item com Scaffolding, selecione Controlador MVC – Vazio>Adicionar.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/ccbdc2b7-c8d0-4aa7-bcd2-3a410dcf0eeb)

Na caixa de diálogo Adicionar Novo Item – MvcMovie, insira HelloWorldController.cs e selecione Adicionar.

Substitua o conteúdo de Controllers/HelloWorldController.cs pelo seguinte código:

~~~C#
using Microsoft.AspNetCore.Mvc;
using System.Text.Encodings.Web;

namespace MvcMovie.Controllers;

public class HelloWorldController : Controller
{
    // 
    // GET: /HelloWorld/
    public string Index()
    {
        return "This is my default action...";
    }
    // 
    // GET: /HelloWorld/Welcome/ 
    public string Welcome()
    {
        return "This is the Welcome action method...";
    }
}

~~~

Cada método public em um controlador pode ser chamado como um ponto de extremidade HTTP. Na amostra acima, ambos os métodos retornam uma cadeia de caracteres. Observe os comentários que precedem cada método.

Um ponto de extremidade HTTP:

É uma URL direcionável no aplicativo Web, como https://localhost:5001/HelloWorld.
Combina:
O protocolo usado: HTTPS.
O local de rede do servidor Web, incluindo a porta TCP: localhost:5001.
O URI de destino: HelloWorld.
O primeiro comentário indica que este é um método HTTP GET invocado por meio do acréscimo de /HelloWorld/ à URL base.

O primeiro comentário especifica um método HTTP GET invocado por meio do acréscimo de /HelloWorld/Welcome/ à URL base. Mais adiante no tutorial, o mecanismo de scaffolding será usado para gerar métodos HTTP POST que atualizam dados.

Execute o aplicativo sem o depurador.

Acrescente /HelloWorld ao caminho na barra de endereços. O método Index retorna uma cadeia de caracteres.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/ef5b5941-2a5f-4e5c-95a0-76742ba4a970)

O MVC invoca as classes do controlador e os métodos de ação dentro delas, dependendo da URL de entrada. A lógica de roteamento de URL padrão usada pelo MVC usa um formato como este para determinar o código a ser invocado:

/[Controller]/[ActionName]/[Parameters]

O formato de roteamento é definido no arquivo Program.cs .

~~~c#
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
~~~

Quando você acessa o aplicativo e não fornece nenhum segmento de URL, ele usa como padrão o controlador “Home” e o método “Index” especificado na linha do modelo realçada acima. Nos segmentos de URL anteriores:

O primeiro segmento de URL determina a classe do controlador a ser executada. 
> Portanto, o localhost:5001/HelloWorld mapeia para a classe HelloWorldController.
A segunda parte do segmento de URL determina o método de ação na classe. 
> Portanto, localhost:5001/HelloWorld/Index faz com que o método Index da classe HelloWorldController seja executado.

Observe que você precisou apenas navegar para localhost:5001/HelloWorld e o método Index foi chamado por padrão.
Index é o método padrão que será chamado em um controlador se não houver um nome de método explicitamente especificado.
A terceira parte do segmento de URL (id) refere-se aos dados de rota. Os dados de rota são explicados posteriormente no tutorial.
Procure: https://localhost:{PORT}/HelloWorld/Welcome. Substitua {PORT} pelo número da porta.

O método Welcome é executado e retorna a cadeia de caracteres This is the Welcome action method.... Para essa URL, o controlador é HelloWorld e Welcome é o método de ação. Você ainda não usou a parte [Parameters] da URL.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/315e6c8e-98bc-4f02-87f5-aca797eeb8ab)

Modifique o código para passar algumas informações de parâmetro da URL para o controlador. Por exemplo, /HelloWorld/Welcome?name=Sandra&numtimes=4.

Altere o método Welcome para incluir dois parâmetros, conforme mostrado no código a seguir.

~~~C#
  // GET: /HelloWorld/Welcome/ 
// Requires using System.Text.Encodings.Web;
public string Welcome(string name, int numTimes = 1)
{
    return HtmlEncoder.Default.Encode($"Hello {name}, NumTimes is: {numTimes}");
}
~~~
O código anterior:

Usa o recurso de parâmetro opcional do C# para indicar que o parâmetro numTimes usa 1 como padrão se nenhum valor é passado para esse parâmetro.
Usa HtmlEncoder.Default.Encode para proteger o aplicativo contra entrada mal-intencionada, como através de JavaScript.
Usa Cadeias de caracteres interpoladas em $"Hello {name}, NumTimes is: {numTimes}".
Execute o aplicativo e navegue até: https://localhost:{PORT}/HelloWorld/Welcome?name=Sandra&numtimes=4. Substitua {PORT} pelo número da porta.

Experimente valores diferentes para name e numtimes na URL. O sistema de model binding do MVC mapeia automaticamente os parâmetros nomeados da cadeia de consulta para os parâmetros no método. Consulte Model binding para obter mais informações.

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/12629aff-1c57-4921-a660-691bf007d6aa)

Na imagem anterior:

O segmento de URL Parameters não é usado.
Os parâmetros name e numTimes são passados na cadeia de caracteres de consulta.
O ? (ponto de interrogação) na URL acima é um separador seguido pela cadeia de consulta.
O caractere & separa os pares campo-valor.
Substitua o método Welcome pelo seguinte código:

~~~C#
public string Welcome(string name, int ID = 1)
{
    return HtmlEncoder.Default.Encode($"Hello {name}, ID: {ID}");
}
~~~~

No exemplo anterior:

O terceiro segmento de URL correspondeu ao parâmetro de rota id.
O método Welcome contém um parâmetro id que correspondeu ao modelo de URL no método MapControllerRoute.
O ? à direita (em id?) indica que o parâmetro id é opcional.
