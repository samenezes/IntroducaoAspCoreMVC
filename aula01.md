# Aula 01 Criar projeto ASP.Net Core MVC

Este tutorial ensina a usar o desenvolvimento Web do ASP.NET Core MVC com controladores(controllers) e exibições(Views).
No final do tutorial, você terá um aplicativo que gerencia e exibe dados de filmes.
Você aprenderá como:
•	Criar um aplicativo Web.
•	Adicionar e gerar o scaffolding de um modelo.
•	Trabalhar com banco de dados.
•	Adicionar pesquisa e validação.


## Criar projeto 
1. •	Inicie o Visual Studio e selecione Criar um projeto.
2. •	Na caixa de diálogo Criar um novo projeto, selecione Aplicativo Web do ASP.NET Core (Model-View-Controller)>Avançar.
3. •	Na caixa de diálogo Configurar seu novo projeto, insira MvcMovie no Nome do projeto. É importante nomear o projeto MvcMovie. O uso de maiúsculas e minúsculas precisa corresponder a cada namespace quando o código é copiado.
4. •	Selecione Avançar.
5. •	Na caixa de diálogo Informações adicionais:
o	Selecionar o .NET 7.0.
o	Verifique se Não usar instruções de nível superior está desmarcado.
6.	Selecione Criar.

 ![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/d0302cde-1413-4297-8453-65afc4cdb7d2)

O Visual Studio usa o modelo de projeto padrão para o projeto MVC criado. O projeto criado:
•	É um aplicativo funcional.
•	Este é um projeto inicial básico.
Executar o aplicativo
Selecione Ctrl+F5 para executar o aplicativo sem o depurador.
O Visual Studio exibe a seguinte caixa de diálogo quando um projeto ainda não está configurado para usar o SSL:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/a966f223-d186-4119-806b-6919fdc4bb6f)

Selecione Sim se você confia no certificado SSL do IIS Express.

A seguinte caixa de diálogo é exibida:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/1058560a-7049-4f4f-9d5f-a8187673bf5f)

Selecione Sim se você concordar com confiar no certificado de desenvolvimento.

Para obter informações sobre como confiar no navegador Firefox, confira Erro de certificado Firefox SEC_ERROR_INADEQUATE_KEY_USAGE.

O Visual Studio executa o aplicativo e abre o navegador padrão.

A barra de endereços mostra localhost:<port#> e não algo como example.com. O nome do host padrão de seu computador local localhost. Uma porta aleatória é usada para o servidor Web quando o Visual Studio cria um projeto Web.

Iniciar o aplicativo sem depuração selecionando Ctrl+F5 permite que você:

Realize alterações de código.
Salve o arquivo.
Atualize rapidamente o navegador e veja as alterações de código.
Você pode iniciar o aplicativo no modo de não depuração ou de depuração por meio do menu Depurar:
![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/8394bbaf-944c-4bb8-893f-d5047ff85a26)

Você pode depurar o aplicativo selecionando o botão https na barra de ferramentas:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/964946ff-ab6e-4992-aa87-3a5b6c56a340)

A imagem a seguir mostra o aplicativo:

![image](https://github.com/samenezes/IntroducaoAspCoreMVC/assets/61150892/801e4628-641c-4b9d-82e6-3b35ee2ae234)



