<properties 
	pageTitle="Políticas no Gerenciamento de API do Azure | Microsoft Azure" 
	description="Aprenda a criar, editar e configurar políticas de Gerenciamento de API." 
	services="api-management" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags 
	ms.service="api-management" 
	ms.workload="mobile" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/12/2016" 
	ms.author="sdanie"/>


#Políticas do Gerenciamento de API do Azure

No Gerenciamento de API do Azure, as políticas são uma poderosa funcionalidade do sistema que permite ao editor alterar o comportamento da API por meio da configuração. As políticas são um conjunto de instruções executadas em sequência, no momento da solicitação ou da resposta de uma API. Instruções populares incluem a conversão do formato de XML para JSON e limite de taxa de chamada para restringir a quantidade de chamadas recebidas de um desenvolvedor. Muitas políticas estão disponíveis pré-configuradas.

Consulte a [Referência de Política][] para ver uma lista completa das instruções de política e suas configurações.

As políticas são aplicadas dentro do gateway que fica entre o consumidor da API e a API gerenciada. O gateway recebe todas as solicitações e normalmente as encaminha inalteradas à API subjacente. No entanto, uma política também pode aplicar mudanças à solicitação de entrada e à resposta de saída.

Expressões de política podem ser usadas como valores de atributo ou texto em qualquer uma das políticas de Gerenciamento de API, a menos que a política especifique o contrário. Algumas políticas, como [Controlar fluxo][] e [Definir variável][] se baseiam em expressões de políticas. Para obter mais informações, consulte [Políticas avançadas][] e [Expressões de política][].

## <a name="scopes"> </a>Como configurar políticas
As políticas podem ser configuradas globalmente ou no escopo de um [Produto][], [API][] ou [Operação][]. Para configurar uma política, navegue até o Editor de políticas no Portal do Editor.

![Menu de políticas][policies-menu]

O editor de políticas consiste em três seções principais: o escopo de política (superior), definição de política, em que as políticas são editadas (à esquerda) e a lista de instruções (à direita):

![Editor de políticas][policies-editor]

Para começar a configurar uma política, você precisa antes selecionar o escopo ao qual ela deverá se aplicar. Na captura de tela abaixo, foi selecionado o produto **Inicial.** Observe que o símbolo de quadrado ao lado do nome da política indica que ela já está aplicada a este nível.

![Escopo][policies-scope]

Uma vez que a política já foi aplicada, a configuração é mostrada na exibição de definição.

![Configurar][policies-configure]

Primeiro, a política é exibida em formato somente leitura. Para editar a definição, clique na ação **Configurar Política**.

![Editar][policies-edit]

A definição da política é um documento XML simples que descreve uma sequência de instruções de entrada e de saída. O XML pode ser editado diretamente na janela de definição. Uma lista de instruções é fornecida à direita e as declarações aplicáveis ao escopo atual ficam habilitadas e destacadas, conforme demonstrado pela instrução **Limit Call Rate** (Taxa limite de chamadas) na captura de tela acima.

Clicar em uma instrução habilitada adicionará o XML adequado ao local onde estiver o cursor na exibição de definição.

>[AZURE.NOTE] Se a política que deseja adicionar não estiver habilitada, verifique se você está no escopo correto para essa política. Cada declaração de política é projetada para uso em determinados escopos e seções da política. Para examinar as seções da política e os escopos de uma política, verifique a seção **Uso** dessa política na [Referência à política][].

Uma lista completa de instruções de políticas e suas configurações está disponível na [Referência de política][].

Por exemplo, para adicionar uma nova instrução para restringir as solicitações de entrada a endereços IP especificados, posicione o cursor dentro do conteúdo do elemento XML `inbound` e clique na instrução **Restringir IPs de chamada**.

![Políticas de restrição][policies-restrict]

Isto adicionará um trecho XML ao elemento `inbound` que fornecerá diretrizes de como configurar a instrução.

	<ip-filter action="allow | forbid">
		<address>address</address>
		<address-range from="address" to="address"/>
	</ip-filter>

Para limitar as solicitações de entrada e aceitar somente as provenientes de um endereço IP 1.2.3.4, modifique o XML da seguinte forma:

	<ip-filter action="allow">
		<address>1.2.3.4</address>
	</ip-filter>

![Salvar][policies-save]

Quando concluir a configuração das instruções da política, clique em **Salvar** para que as alterações sejam propagadas para o gateway de Gerenciamento de API imediatamente.

##<a name="sections"> </a>Compreendendo configuração de políticas

Uma política é uma série de instruções que são executadas para uma solicitação e uma resposta. A configuração é dividida adequadamente entre as seções `inbound`, `backend`, `outbound` e `on-error`, conforme demonstrado na configuração seguinte.

	<policies>
	  <inbound>
	    <!-- statements to be applied to the request go here -->
	  </inbound>
	  <backend>
	    <!-- statements to be applied before the request is forwarded to 
	         the backend service go here -->
	  </backend>
	  <outbound>
	    <!-- statements to be applied to the response go here -->
	  </outbound>
	  <on-error>
	    <!-- statements to be applied if there is an error condition go here -->
	  </on-error>
	</policies> 

Se houver um erro durante o processamento de uma solicitação, quaisquer etapas restantes nas seções `inbound`, `backend` ou `outbound` serão ignoradas e a execução saltará para as instruções na seção `on-error`. Ao colocar instruções de políticas na seção `on-error`, você pode revisar o erro usando a propriedade `context.LastError`, inspecionar e personalizar a resposta de erro usando a política `set-body` e configurar o que acontece se ocorrer um erro. Há códigos de erro para obter as etapas internas e erros que podem ocorrer durante o processamento de instruções de política. Para obter mais informações, consulte [Tratamento de erros em políticas de gerenciamento de API](https://msdn.microsoft.com/library/azure/mt629506.aspx).

Uma vez que as políticas podem ser especificadas em diferentes níveis (global, de produto, API e operação), a configuração oferece uma forma de especifica a ordem na qual as instruções dessa definição são executadas com relação à política pai.

Os escopos de política são avaliados na ordem a seguir.

1. Escopo global
2. Escopo do produto
3. Escopo de API
4. Escopo de operação

As declarações dentro deles são avaliadas de acordo com o posicionamento do elemento `base`, se ele estiver presente.

Por exemplo, se você tiver uma política a nível global e uma política configurada para uma API, então, sempre que essa API em particular for usado, ambas as políticas serão aplicadas. O Gerenciamento de API permite uma ordenação determinista de instruções de política combinadas por meio do elemento base.

	<policies>
    	<inbound>
        	<cross-domain />
        	<base />
        	<find-and-replace from="xyz" to="abc" />
    	</inbound>
	</policies>

No exemplo de definição de política acima, a instrução `cross-domain` seria executada antes de quaisquer políticas maiores que, por sua vez, seriam seguidas da política `find-and-replace`.

Se a mesma política aparece duas vezes na declaração de política, a política avaliada mais recentemente é aplicada. Você pode usar isso para substituir políticas que são definidas em um escopo maior. Para ver as políticas no escopo atual no editor de política, clique em **Recalcular a política efetiva para o escopo selecionado**.

Observe que uma política global não tem nenhuma política pai e que usar o elemento `<base>` nela não tem nenhum efeito.

## Próximas etapas

Confira o vídeo a seguir sobre expressões de política.

> [AZURE.VIDEO policy-expressions-in-azure-api-management]

[Referência de Política]: api-management-policy-reference.md
[Referência à política]: api-management-policy-reference.md
[Produto]: api-management-howto-add-products.md
[API]: api-management-howto-add-products.md#add-apis
[Operação]: api-management-howto-add-operations.md

[Políticas avançadas]: https://msdn.microsoft.com/library/azure/dn894085.aspx
[Controlar fluxo]: https://msdn.microsoft.com/library/azure/dn894085.aspx#choose
[Definir variável]: https://msdn.microsoft.com/library/azure/dn894085.aspx#set_variable
[Expressões de política]: https://msdn.microsoft.com/library/azure/dn910913.aspx

[policies-menu]: ./media/api-management-howto-policies/api-management-policies-menu.png
[policies-editor]: ./media/api-management-howto-policies/api-management-policies-editor.png
[policies-scope]: ./media/api-management-howto-policies/api-management-policies-scope.png
[policies-configure]: ./media/api-management-howto-policies/api-management-policies-configure.png
[policies-edit]: ./media/api-management-howto-policies/api-management-policies-edit.png
[policies-restrict]: ./media/api-management-howto-policies/api-management-policies-restrict.png
[policies-save]: ./media/api-management-howto-policies/api-management-policies-save.png

<!---HONumber=AcomDC_0413_2016-->