<properties
	pageTitle="Introdução ao AngularJS do Azure AD v2.0 | Microsoft Azure"
	description="Como compilar um aplicativo de página única Angular JS que autentica usuários com contas da Microsoft pessoais e contas corporativas ou de estudante."
	services="active-directory"
	documentationCenter=""
	authors="dstrockis"
	manager="mbaldwin"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="javascript"
	ms.topic="article"
	ms.date="02/20/2016"
	ms.author="dastrock"/>


# Adicionar as credenciais a um aplicativo de página única do AngularJS - NodeJS

Neste artigo, adicionaremos entrada com contas da plataforma Microsoft a um aplicativo AngularJS usando o ponto de extremidade v2.0 do Azure Active Directory. O ponto de extremidade v2.0 permite realizar uma integração única em seu aplicativo e autenticar usuários com contas pessoais e corporativas/de estudante.

Este exemplo é um aplicativo de página única de uma simples lista de tarefas pendentes que armazena as tarefas em uma API REST back-end. Ele é escrito em NodeJS e protegido usando tokens de portador OAuth do AD do Azure. O aplicativo AngularJS usará nossa biblioteca de autenticação JavaScript de código aberto [adal.js](https://github.com/AzureAD/azure-activedirectory-library-for-js) para manipular todo o processo de entrada e adquirir tokens para chamar a API REST. O mesmo padrão pode ser aplicado para autenticar as outras APIs REST, como o [Microsoft Graph](https://graph.microsoft.com) ou as APIs do Gerenciador de Recursos do Azure.

> [AZURE.NOTE]
	Nem todos os recursos e cenários do Azure Active Directory têm suporte no ponto de extremidade v2.0. Para determinar se você deve usar o ponto de extremidade v2.0, leia sobre as [limitações da v2.0](active-directory-v2-limitations.md).

## Baixar

Para começar, será necessário baixar e instalar o [Node.js](https://nodejs.org). Em seguida, você pode clonar ou [baixar](https://github.com/AzureADQuickStarts/AppModelv2-SinglePageApp-AngularJS-NodeJS/archive/skeleton.zip) um aplicativo de esqueleto:

```
git clone --branch skeleton https://github.com/AzureADQuickStarts/AppModelv2-SinglePageApp-AngularJS-NodeJS.git
```

O aplicativo de esqueleto inclui todo o código clichê de um aplicativo AngularJS simples, mas não contém as partes relacionadas à identidade. Se você não quiser acompanhar, clone ou [baixe](https://github.com/AzureADQuickStarts/AppModelv2-SinglePageApp-AngularJS-NodeJS/archive/complete.zip) o exemplo completo.

```
git clone https://github.com/AzureADSamples/SinglePageApp-AngularJS-NodeJS.git
```

## Registrar um aplicativo

Primeiro, crie um aplicativo no [Portal de Registro de Aplicativos](https://apps.dev.microsoft.com), ou execute estas [etapas detalhadas](active-directory-v2-app-registration.md). Não se esqueça de:

- Adicionar a plataforma **Web** para seu aplicativo.
- Inserir o **URI de Redirecionamento** correto. O padrão para esse exemplo é `http://localhost:8080`
- Deixe a caixa de seleção **Permitir Fluxo Implícito** habilitada. 

Copie a **ID do Aplicativo** designada ao seu aplicativo, você precisará dela em breve.

## Instalar o adal.js
Para começar, navegue até o projeto do qual você fez o download e instale o adal.js. Se o [bower](http://bower.io/) estiver instalado, basta executar este comando. Para qualquer incompatibilidade de versão da dependência, basta escolha a versão mais recente.
```
bower install adal-angular#experimental
```

Como alternativa, você pode baixar manualmente o [adal.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/experimental/dist/adal.min.js) e o [adal-angular.js](https://raw.githubusercontent.com/AzureAD/azure-activedirectory-library-for-js/experimental/dist/adal-angular.min.js). Adicione os dois arquivos ao diretório `app/lib/adal-angular-experimental/dist`.

Abra o projeto em seu editor de texto favorito e carregue o adal.js no final do corpo da página:

```html
<!--index.html-->

...

<script src="App/bower_components/dist/adal.min.js"></script>
<script src="App/bower_components/dist/adal-angular.min.js"></script>

...
```

## Configurar a API REST

Enquanto configuramos as coisas, vamos fazer a API REST back-end funcionar. Em um prompt de comando, instale todos os pacotes necessários executando (verifique se você está no diretório de nível superior do projeto):

```
npm install
```

Agora, abra `config.js` e substitua o valor `audience`:

```js
exports.creds = {
     
     // TODO: Replace this value with the Application ID from the registration portal
     audience: '<Your-application-id>',
	 
	 ...
}
```

A API REST usará esse valor para validar tokens que recebe do aplicativo Angular em solicitações AJAX. Observe que essa API REST simples armazena dados na memória. Portanto, cada vez que você parar o servidor, perderá todas as tarefas criadas anteriormente.

Esse é todo o tempo que usaremos para discutir o funcionamento da API REST. Fique à vontade para examinar o código, mas se você quiser saber mais sobre como proteger as APIs Web com o AD do Azure, confira [este artigo](active-directory-v2-devquickstarts-node-api.md).

## Entrada de usuários
Hora de escrever um pouco de código de identidade. Talvez você já tenha notado que o adal.js contém um provedor AngularJS, que funciona perfeitamente com o mecanismo de roteamento Angular. Comece adicionando o módulo adal ao aplicativo:

```js
// app/scripts/app.js

angular.module('todoApp', ['ngRoute','AdalAngular'])
.config(['$routeProvider','$httpProvider', 'adalAuthenticationServiceProvider',
 function ($routeProvider, $httpProvider, adalProvider) {

...
```

Agora, você pode inicializar o `adalProvider` com a ID do Aplicativo:

```js
// app/scripts/app.js

...

adalProvider.init({
        
        // Use this value for the public instance of Azure AD
        instance: 'https://login.microsoftonline.com/', 
        
        // The 'common' endpoint is used for multi-tenant applications like this one
        tenant: 'common',
        
        // Your application id from the registration portal
        clientId: '<Your-application-id>',
        
        // If you're using IE, uncommment this line - the default HTML5 sessionStorage does not work for localhost.
        //cacheLocation: 'localStorage',
         
    }, $httpProvider);
```

Ótimo, agora o adal.js tem todas as informações necessárias para proteger seu aplicativo e autenticar os usuários. Para forçar a entrada por uma rota específica no aplicativo, basta uma linha de código:

```js
// app/scripts/app.js

...

}).when("/TodoList", {
    controller: "todoListCtrl",
    templateUrl: "/static/views/TodoList.html",
    requireADLogin: true, // Ensures that the user must be logged in to access the route
})

...
```

Quando um usuário clica no link `TodoList`, o adal.js o redireciona automaticamente ao AD do Azure para conexão, se for necessário. Também é possível enviar de forma explícita as solicitações de entrada e saída chamando o adal.js em seus controladores:

```js
// app/scripts/homeCtrl.js

angular.module('todoApp')
// Load adal.js the same way for use in controllers and views   
.controller('homeCtrl', ['$scope', 'adalAuthenticationService','$location', function ($scope, adalService, $location) {
    $scope.login = function () {
        
        // Redirect the user to sign in
        adalService.login();
        
    };
    $scope.logout = function () {
        
        // Redirect the user to log out    
        adalService.logOut();
    
    };
...
```

## Exibir informações do usuário
Agora que o usuário está conectado, provavelmente você precisará acessar os dados de autenticação do usuário conectado em seu aplicativo. O Adal.js expõe essas informações para você no objeto `userInfo`. Para acessar este objeto em uma exibição, primeiro adicione adal.js ao escopo raiz do controlador correspondente:

```js
// app/scripts/userDataCtrl.js

angular.module('todoApp')
// Load ADAL for use in view
.controller('userDataCtrl', ['$scope', 'adalAuthenticationService', function ($scope, adalService) {}]);
```

Em seguida, você pode endereçar diretamente o objeto `userInfo` em seu modo de exibição:

```html
<!--app/views/UserData.html-->

...

    <!--Get the user's profile information from the ADAL userInfo object-->
    <tr ng-repeat="(key, value) in userInfo.profile">
        <td>{{key}}</td>
        <td>{{value}}</td>
    </tr>
...
```

Você também pode usar o objeto `userInfo` para determinar se o usuário está conectado ou não.

```html
<!--index.html-->

...

    <!--Use the ADAL userInfo object to show the right login/logout button-->
    <ul class="nav navbar-nav navbar-right">
        <li><a class="btn btn-link" ng-show="userInfo.isAuthenticated" ng-click="logout()">Logout</a></li>
        <li><a class="btn btn-link" ng-hide="userInfo.isAuthenticated" ng-click="login()">Login</a></li>
    </ul>
...
```

## Chamar a API REST
Por fim, é hora de obter alguns tokens e chamar a API REST para criar, ler, atualizar e excluir tarefas. E sabe o que mais? Você não precisa fazer *coisa alguma*. O Adal.js se encarregará automaticamente de obter, armazenar em cache e atualizar os tokens. Ele também se encarregará de conectar esses tokens às solicitações AJAX de saída que você envia à API REST.

Como isso funciona exatamente? É tudo graças à mágica dos [interceptores AngularJS](https://docs.angularjs.org/api/ng/service/$http), que permitem ao adal.js transformar as mensagens http de entrada e de saída. Além disso, o adal.js pressupõe que todas as solicitações enviadas ao mesmo domínio que a janela devem usar tokens destinados à mesma ID de Aplicativo que o aplicativo AngularJS. É por isso que usamos a mesma ID de Aplicativo no aplicativo Angular e na API REST NodeJS. Claro, você pode substituir esse comportamento e instruir o adal.js a obter tokens para outras APIs REST, se for necessário. Mas para este cenário simples, basta os padrão.

Veja um trecho que mostra como é fácil enviar solicitações com tokens de portador do AD do Azure:

```js
// app/scripts/todoListSvc.js

...
return $http.get('/api/tasks');
...
```

Parabéns! Seu aplicativo de página única integrado ao Azure AD está concluído. Vá em frente, receba os aplausos. Ele pode autenticar os usuários, chamar com segurança sua API REST back-end usando OpenID Connect e obter informações básicas sobre o usuário. Para uso imediato, ele oferece suporte a qualquer usuário com uma conta pessoal da Microsoft ou uma conta corporativa/de estudante do AD do Azure. Experimente o aplicativo executando:

```
node server.js
```

Em um navegador, navegue até `http://localhost:8080`. Entre usando uma conta pessoal da Microsoft ou uma conta corporativa/de estudante. Adicione tarefas à lista de tarefas pendentes do usuário e saia. Tente entrar com o outro tipo de conta. Se você precisar de um locatário do AD do Azure a fim de criar usuários corporativos/estudantes, [saiba como obter um aqui](active-directory-howto-tenant.md) (é gratuito).

Para continuar aprendendo sobre o ponto de extremidade v2.0, retorne ao nosso [guia do desenvolvedor v2.0](active-directory-appmodel-v2-overview.md). Para obter recursos adicionais, consulte:

- [Exemplos do Azure no GitHub >>](https://github.com/Azure-Samples)
- [AD do Azure no Excedente de Pilha >>](http://stackoverflow.com/questions/tagged/azure-active-directory)
- Documentação do Azure AD no [Azure.com >>](https://azure.microsoft.com/documentation/services/active-directory/)

<!---HONumber=AcomDC_0323_2016-->