<properties
	pageTitle="Introdução à autenticação para aplicativos móveis no Xamarin iOS"
	description="Aprenda a usar os aplicativos móveis para autenticar usuários de seu aplicativo Xamarin iOS por meio de uma variedade de provedores de identidade, incluindo AAD, Google, Facebook, Twitter e Microsoft."
	services="app-service\mobile"
	documentationCenter="xamarin"
	authors="mattchenderson"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="app-service-mobile"
	ms.workload="na"
	ms.tgt_pltfrm="mobile-xamarin-ios"
	ms.devlang="dotnet"
	ms.topic="article"
	ms.date="02/04/2016"
	ms.author="mahender"/>

# Adicionar autenticação ao aplicativo Xamarin.iOS

[AZURE.INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

Este tópico mostra como autenticar usuários de um aplicativo móvel do Serviço de Aplicativo em seu aplicativo cliente. Neste tutorial, você adiciona a autenticação ao projeto de início rápido do Xamarin.iOS usando um provedor de identidade com suporte do Serviço de Aplicativo. Depois de ser autenticado e autorizado com êxito pelo Aplicativo Móvel, o valor da ID de usuário é exibido e você poderá acessar dados da tabela restrita.

Você deve primeiro concluir o tutorial [Criar um aplicativo Xamarin.iOS]. Se você não usar o projeto baixado de início rápido do servidor, deve adicionar o pacote de extensão de autenticação ao seu projeto. Para obter mais informações sobre pacotes de extensão do servidor, confira [Trabalhar com o servidor .NET back-end do SDK para Aplicativos Móveis do Azure](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md).

##Registrar seu aplicativo para a autenticação e configurar os Serviços de Aplicativos

[AZURE.INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

##Restringir permissões a usuários autenticados

[AZURE.INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

&nbsp;&nbsp;4. No Visual Studio ou Xamarin Studio, execute o projeto cliente em um dispositivo ou emulador. Verifique se uma exceção não tratada com um código de status 401 (Não autorizado) é gerada após o aplicativo ser iniciado. A falha será registrada no console do depurador. Então no Visual Studio, você deve ver a falha na janela de saída.

&nbsp;&nbsp;Essa falha não autorizada acontece porque o aplicativo tenta acessar o back-end do aplicativo móvel como um usuário não autenticado. A tabela *TodoItem* agora exige autenticação.

Em seguida, você atualizará o aplicativo do cliente para solicitar recursos do back-end do aplicativo móvel com um usuário autenticado.

##Adicionar autenticação ao aplicativo

Nesta seção, você modificará o aplicativo para exibir uma tela de logon antes de exibir os dados. Quando o aplicativo for iniciado, ele não se conectará ao serviço de aplicativo e não exibirá nenhum dado. Depois que o usuário executar pela primeira vez um gesto de atualização, a tela de logon aparecerá e, após o êxito no logon, a lista de itens de tarefas pendentes será exibida.

1. No projeto do cliente, abra o arquivo **QSTodoService.cs** e adicione a seguinte instrução using e `MobileServiceUser` com acessador à classe QSTodoService:

	```
		using UIKit;
	```

		// Logged in user
		private MobileServiceUser user;
		public MobileServiceUser User { get { return user; } }

2. Adicione um novo método chamado **Authenticate** ao **QSTodoService** com a seguinte definição:


        public async Task Authenticate(UIViewController view)
        {
            try
            {
                user = await client.LoginAsync(view, MobileServiceAuthenticationProvider.Facebook);
            }
            catch (Exception ex)
            {
                Console.Error.WriteLine (@"ERROR - AUTHENTICATION FAILED {0}", ex.Message);
            }
        }

	>[AZURE.NOTE] Se você estiver usando um provedor de identidade que não seja o Facebook, altere o valor passado para **LoginAsync** acima para um dos seguintes: _MicrosoftAccount_, _Twitter_, _Google_ ou _WindowsAzureActiveDirectory_.

3. Abra o **QSTodoListViewController.cs**. Modifique a definição do método de **ViewDidLoad** removendo a chamada para **RefreshAsync()** perto do final:

		public override async void ViewDidLoad ()
		{
			base.ViewDidLoad ();

			todoService = QSTodoService.DefaultService;
           await todoService.InitializeStoreAsync ();

           RefreshControl.ValueChanged += async (sender, e) => {
                await RefreshAsync ();
           }

			// Comment out the call to RefreshAsync
			// await RefreshAsync ();
		}


4. Modifique o método **RefreshAsync** se a propriedade **User** for null. Adicione o seguinte código à parte superior da definição do método:

		// start of RefreshAsync method
		if (todoService.User == null) {
			await QSTodoService.DefaultService.Authenticate (this);
			if (todoService.User == null) {
				Console.WriteLine ("couldn't login!!");
				return;
			}
		}
		// rest of RefreshAsync method

5. No Visual Studio ou Xamarin Studio conectado ao seu Host de compilação Xamarin em seu Mac, execute o projeto de cliente direcionado a um dispositivo ou emulador. Verifique se o aplicativo não exibe dados.

	Faça um gesto de atualização pressionando a lista de itens, o que fará com que a tela de logon apareça. Depois de fornecer credenciais válidas, o aplicativo exibirá a lista de itens de tarefas e você poderá atualizar os dados.


<!-- URLs. -->
[Submit an app page]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Criar um aplicativo Xamarin.iOS]: app-service-mobile-xamarin-ios-get-started.md

<!---HONumber=AcomDC_0211_2016-->