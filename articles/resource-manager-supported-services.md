<properties
   pageTitle="Serviços, regiões, esquemas e versões com suporte do Gerenciador de Recursos | Microsoft Azure"
   description="Descreve os provedores de recursos que oferecem suporte ao Gerenciador de Recursos, aos respectivos esquemas e versões de API disponíveis, bem como às regiões que podem hospedar os recursos."
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor="tysonn"/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="03/01/2016"
   ms.author="tomfitz"/>

# Provedores, regiões, versões de API e esquemas do Gerenciador de Recursos

O Gerenciador de Recursos do Azure fornece uma nova maneira de implantar e gerenciar os serviços que compõem seus aplicativos. A maioria (não todos) dos serviços suporta o Gerenciador de Recursos e alguns serviços suportam o Gerenciador de Recursos apenas parcialmente. A Microsoft habilitará o Gerenciador de Recursos para todos os serviços importantes para soluções futuras, mas, até que o suporte seja consistente, você precisa saber o status atual de cada serviço. Este tópico fornece uma lista de provedores de recursos com suporte para o Gerenciador de Recursos do Azure.

Ao implantar seus recursos, você também precisa saber quais são as regiões que oferecem suporte a esses recursos e quais versões de API estão disponíveis para os recursos. A seção [Regiões com suporte](#supported-regions) mostra como descobrir quais regiões funcionarão para sua assinatura e seus recursos. A seção [Versões de API com suporte](#supported-api-versions) mostra como determinar quais versões de API você pode usar.

Para ver quais serviços têm suporte no Portal do Azure e no portal clássico, confira o [Gráfico de disponibilidade do Portal do Azure](https://azure.microsoft.com/features/azure-portal/availability/). Para ver quais serviços oferecem suporte a recursos de movimentação, confira [Mover recursos para o novo grupo de recursos ou assinatura](resource-group-move-resources.md).

As tabelas a seguir listam quais serviços suportam a implantação e o gerenciamento por meio do Gerenciador de Recursos e quais não. O link na coluna **Modelos de Início Rápido** envia uma consulta ao repositório Modelos de Início Rápido do Azure para o provedor de recursos especificado. Os modelos de início rápido são adicionados e atualizados com frequência. A presença de um link para um serviço específico não significa necessariamente que a consulta retornará modelos do repositório.


## Computação

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------------------------ |-------- | ------ | ------ |
| Batch | Sim | [REST do Lote](https://msdn.microsoft.com/library/azure/dn820158.aspx) | | [Microsoft.Batch](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Batch%22&type=Code) |
| Serviços de ciclo de vida do Dynamics | Sim | | | [Microsoft.DynamicsLcs](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.DynamicsLcs%22&type=Code)
| Service Fabric (visualização) | Sim | [Service Fabric Rest](https://msdn.microsoft.com/library/azure/dn707692.aspx) | | [Microsoft.ServiceFabric](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.ServiceFabric%22&type=Code) |
| Máquinas Virtuais | Sim | [VM REST](https://msdn.microsoft.com/library/azure/mt163647.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Compute.json) | [Microsoft.Compute](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Compute%22&type=Code) |
| Máquinas virtuais (clássico) | Limitado | - | - |
 Aplicativo Remoto | Não | - | - |
 | Serviços de Nuvem (clássico) | Limitado (veja abaixo) | - | - | - |

Máquinas virtuais (clássico) refere-se aos recursos que foram implantados por meio do modelo de implantação clássico, não por meio do modelo de implantação do Gerenciador de Recursos. Em geral, esses recursos não oferecem suporte a operações do Gerenciador de Recursos, mas existem algumas operações que foram habilitadas. Para saber mais sobre esses modelos de implantação, confira [Noções básicas sobre a implantação do Gerenciador de Recursos e a implantação clássica](resource-manager-deployment-model.md).

Os Serviços de Nuvem (clássico) podem ser usados com outros recursos clássicos; no entanto, os recursos clássicos não tiram proveito de todos os recursos do Gerenciador de Recursos e não são uma boa opção para soluções futuras. Em vez disso, considere alterar a infraestrutura de aplicativos para usar os recursos dos namespaces Microsoft.Compute, Microsoft.Storage e Microsoft.Network.


## Rede

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | -------  | -------- | ------ | ------ |
| Application Gateway | Sim | | | [applicationGateways](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2FapplicationGateways%22&type=Code) |
| DNS | Sim | [DNS REST](https://msdn.microsoft.com/library/azure/mt163862.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) | [dnsZones](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2FdnsZones%22&type=Code) |
| Rota Expressa | Sim | [REST da Rota Expressa](https://msdn.microsoft.com/library/azure/mt586720.aspx) | | [expressRouteCircuits](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2FexpressRouteCircuits%22&type=Code) |
| Balanceador de carga | Sim | [REST do balanceador de carga](https://msdn.microsoft.com/library/azure/mt163651.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) | [loadBalancers](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2Floadbalancers%22&type=Code) |
| Gerenciador de Tráfego | Sim | [REST do Gerenciador de Tráfego](https://msdn.microsoft.com/library/azure/mt163667.aspx) | [2015-11-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-11-01/Microsoft.Network.json) | [trafficmanagerprofiles](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2Ftrafficmanagerprofiles%22&type=Code) |
| Redes Virtuais | Sim| [REST da Rede Virtual](https://msdn.microsoft.com/pt-BR/library/azure/mt163650.aspx) | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Network.json) | [virtualNetworks](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2FvirtualNetworks%22&type=Code) |
| Gateway de VPN | Sim | [REST de Gateway de rede](https://msdn.microsoft.com/library/azure/mt163859.aspx) | | [conexões](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2Fconnections%22&type=Code) [virtualNetworkGateways](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2FvirtualNetworkGateways%22&type=Code) <br /> [localNetworkGateways](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Network%2FlocalNetworkGateways%22&type=Code) <br /> |



## Armazenamento de dados

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------- | ------ |
| Banco de Dados de Documentos | Sim | [REST do Banco de Dados de Documentos](https://msdn.microsoft.com/library/azure/dn781481.aspx) | [2015-04-08](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-04-08/Microsoft.DocumentDB.json) | [Microsoft.DocumentDB](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.DocumentDb%22&type=Code) |
| Cache Redis | Sim | | [2014-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Cache.json) | [Microsoft.Cache](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Cache%22&type=Code) |
| Pesquisar | Sim | [REST de pesquisa](https://msdn.microsoft.com/library/azure/dn798935.aspx) | | [Microsoft.Search](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Search%22&type=Code) |
| Armazenamento | Sim | [REST de armazenamento](https://msdn.microsoft.com/library/azure/mt163683.aspx) | [Conta de armazenamento](resource-manager-template-storage.md) | [Microsoft.Storage](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Storage%22&type=Code) |
| Banco de dados SQL | Sim | [REST do Banco de Dados SQL](https://msdn.microsoft.com/library/azure/mt163571.aspx) | [2014-04-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01-preview/Microsoft.Sql.json) | [Microsoft.Sql](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Sql%22&type=Code) |
| SQL Data Warehouse | Sim | | |
| StorSimple | Não | - | - | - | - |

## Web e serviços móveis

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------ |
| Aplicativos de API | Sim | | [2015-03-01-preview](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-03-01-preview/Microsoft.AppService.json) | [Aplicativos de API](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22kind%22%3A+%22apiApp%22&type=Code) |
| Gerenciamento da API | Sim | [REST de gerenciamento de API](https://msdn.microsoft.com/library/azure/dn776326.aspx) | | [Microsoft.ApiManagement](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.ApiManagement%22&type=Code) | 
| Aplicativos Lógicos | Sim | | | [Microsoft.Logic](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Logic%22&type=Code) |
| Aplicativos Móveis | Sim | | | |
| Compromisso de mobilidade | Sim | | | [Microsoft.MobileEngagements](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.MobileEngagement%22&type=Code) |
| Aplicativos Web | Sim | | [2015-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-08-01/Microsoft.Web.json) | [Microsoft.Web](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Web%22&type=Code) |

## Análise

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | -------  | -------- | ------ | ------ |
| Data Factory | Sim | [REST do Data Factory](https://msdn.microsoft.com/library/azure/dn906738.aspx) | | [Microsoft.DataFactory](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.DataFactory%22&type=Code) |
| Análises Data Lake | Sim | | | |
| Repositório Data Lake | Sim | | | |
| HDInsights | Sim | [REST do HDInsights](https://msdn.microsoft.com/library/azure/mt622197.aspx) | | [Microsoft.HDInsight](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.HDInsight%22&type=Code) |
| Análise de fluxo | Sim | [Análise de fluxo REST](https://msdn.microsoft.com/library/azure/dn835031.aspx) | | [Microsoft.StreamAnalytics](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.StreamAnalytics%22&type=Code) |
| Aprendizado de Máquina | Não | - | - | - |
| Catálogo de Dados | Não | - | - | - |

## Internet das coisas

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------ |
| Hub de evento | Sim | [REST do Hub de Eventos](https://msdn.microsoft.com/library/azure/dn790674.aspx) | | [Microsoft.EventHub](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.EventHub%22&type=Code) |
| IoTHubs | Sim | [REST do Hub IoT](https://msdn.microsoft.com/library/azure/mt589014.aspx) | | [Microsoft.Devices](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Devices%22&type=Code) |
| Hubs de Notificação | Sim | [REST do Hub de notificação](https://msdn.microsoft.com/library/azure/dn495827.aspx) | [2015-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-04-01/Microsoft.NotificationHubs.json) | [Microsoft.NotificationHubs](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.NotificationHubs%22&type=Code) |

## Mídia e CDN

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------ |
| CDN | Sim | [REST CDN](https://msdn.microsoft.com/library/azure/mt634456.aspx) | | [Microsoft.Cdn](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Cdn%22&type=Code) |
| Serviço de mídia | Não | | |


## Integração híbrida

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------ |
| Serviços do BizTalk | Sim | | [2014-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01/Microsoft.BizTalkServices.json) | [Microsoft.BizTalkServices](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.BizTalkServices%22&type=Code) |
| Barramento de Serviço | Sim | | | [Microsoft.ServiceBus](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.ServiceBus%22&type=Code) |
| Backup | Não | - | - | - |
| Recuperação de Site | Não | - | - | - |

## Gerenciamento de acesso e identidade 

O Active Directory do Azure funciona com o Gerenciador de Recursos para habilitar o controle de acesso baseado em função para sua assinatura. Para obter mais informações sobre o controle de acesso baseado em função e Active Directory, confira [Controle de Acesso Baseado em Função do Azure](./active-directory/role-based-access-control-configure.md).

## Serviços para Desenvolvedores 

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------ |
| Application Insights | Sim | | [2014-04-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-04-01/Microsoft.Insights.json) | [Microsoft.insights](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.insights%22&type=Code) |
| Bing Mapas | Sim | | | [Microsoft.BingMaps](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.BingMaps%22&type=Code) |
| Laboratórios de Desenvolvimento/Teste (visualização) | Sim | | [2015-05-21-visualização](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2015-05-21-preview/Microsoft.DevTestLab.json) | [Microsoft.DevTestLab](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.DevTestLab%22&type=Code) |
| Conta do Visual Studio | Sim | | [2014-02-26](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-02-26/microsoft.visualstudio.json) | [Microsoft.VisualStudio](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.VisualStudio%22&type=Code) |

## Gerenciamento e segurança

| O Barramento de | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------- | ------ | ------ |
| Automação | Sim | | | [Microsoft.Automation](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Automation%22&type=Code) |
| Cofre da Chave | Sim | [REST do Cofre da Chave](https://msdn.microsoft.com/library/azure/dn903609.aspx) | [Cofre da chave](resource-manager-template-keyvault.md)<br />[Segredo do cofre da chave](resource-manager-template-keyvault-secret.md) | [Microsoft.KeyVault](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.KeyVault%22&type=Code) |
| Insights Operacionais | Sim | | | [Microsoft.OperationalInsights](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.OperationalInsights%22&type=Code) |
| Agendador | Sim | [REST do Agendador](https://msdn.microsoft.com/library/azure/mt629143.aspx) | [2014-08-01](https://github.com/Azure/azure-resource-manager-schemas/blob/master/schemas/2014-08-01/Microsoft.Scheduler.json) | [Microsoft.Scheduler](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Scheduler%22&type=Code) |
| Segurança (visualização) | Sim | | | [Microsoft.Security](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Security%22&type=Code) |

## Gerenciador de Recursos

| Recurso | Gerenciador de Recursos habilitado | API REST | Esquema | Modelos de Início Rápido |
| ------- | ------- | -------------- | -------- | ------ | ------ |
| Autorização | Sim | [Gerenciamento de bloqueios](https://msdn.microsoft.com/library/azure/mt204563.aspx)<br >[Controle de acesso baseado em função](https://msdn.microsoft.com/library/azure/dn906885.aspx) | [Bloqueio de recurso](resource-manager-template-lock.md)<br />[Atribuições de função](resource-manager-template-role.md) | [Microsoft.Authorization](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Authorization%22&type=Code) |
| Recursos | Sim | [Recursos vinculados](https://msdn.microsoft.com/library/azure/mt238499.aspx) | [Link de recursos](resource-manager-template-links.md) | [Microsoft.Resources](https://github.com/Azure/azure-quickstart-templates/search?utf8=%E2%9C%93&q=%22Microsoft.Resources%22&type=Code) |


## Provedores e tipos de recursos

Ao implantar recursos, com frequência você precisa recuperar informações sobre os provedores e tipos de recursos. Você pode recuperar essas informações por meio da API REST, do Azure PowerShell ou da CLI do Azure.

Para trabalhar com um provedor de recursos, o provedor de recursos deve ser registrado com sua conta. Por padrão, vários provedores de recursos são automaticamente registrados; no entanto, talvez seja necessário registrar manualmente alguns provedores de recursos. Os exemplos abaixo mostram como obter o status do registro de um provedor de recursos e registrar o provedor de recursos, se necessário.

### API REST

Para obter todos os provedores de recursos disponíveis, incluindo seus tipos, locais, versões de API e status do registro, use a operação [Listar todos os provedores de recursos](https://msdn.microsoft.com/library/azure/dn790524.aspx). Se precisar registrar um provedor de recursos, veja [Registrar uma assinatura em um provedor de recursos](https://msdn.microsoft.com/library/azure/dn790548.aspx).

### PowerShell

O exemplo a seguir mostra como obter todos os provedores de recursos disponíveis.

    Get-AzureRmResourceProvider -ListAvailable
    
A saída será semelhante a:

    ProviderNamespace               RegistrationState ResourceTypes
    -----------------               ----------------- -------------
    Microsoft.ApiManagement         Unregistered      {service, validateServiceName, checkServiceNameAvailability}
    Microsoft.AppService            Registered        {apiapps, appIdentities, gateways, deploymenttemplates...}
    ...

O exemplo a seguir mostra como obter os tipos de recursos para um provedor de recursos específico.

    (Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes
    
A saída será semelhante a:

    ResourceTypeName                Locations                                         ApiVersions
    ----------------                ---------                                         ------
    sites/extensions                {Brazil South, East Asia, East US, Japan East...} {20...
    sites/slots/extensions          {Brazil South, East Asia, East US, Japan East...} {20...
    ...
    
Para registrar um provedor de recursos, forneça o namespace:

    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ApiManagement

### CLI do Azure

O exemplo a seguir mostra como obter todos os provedores de recursos disponíveis.

    azure provider list
    
A saída será semelhante a:

    info:    Executing command provider list
    + Getting ARM registered providers
    data:    Namespace                        Registered
    data:    -------------------------------  -------------
    data:    Microsoft.ApiManagement          Unregistered
    data:    Microsoft.AppService             Registered
    data:    Microsoft.Authorization          Registered
    ...

Você pode salvar as informações para um provedor de recursos específico em um arquivo com o comando a seguir.

    azure provider show Microsoft.Web -vv --json > c:\temp.json

Para registrar um provedor de recursos, forneça o namespace:

    azure provider register -n Microsoft.ServiceBus

## Regiões com suporte

Durante a implantação de recursos, você normalmente precisa especificar uma região para os recursos. O Gerenciador de Recursos tem suporte em todas as regiões, mas os recursos que você implanta talvez não tenham suporte em todas as regiões. Além disso, pode haver limitações em sua assinatura que impedem o uso de algumas regiões que oferecem suporte aos recursos. Essas limitações podem estar relacionadas a problemas tributários de seu país de residência ou ao resultado de uma política colocada pelo seu administrador de assinatura para usar somente determinadas regiões.

Para obter uma lista completa de todas as regiões com suporte para todos os serviços do Azure, veja [Serviços por região](https://azure.microsoft.com/regions/#services); no entanto, essa lista pode incluir regiões que não tem suporte em sua assinatura. É possível determinar as regiões para um determinado tipo de recurso com suporte da sua assinatura executando um dos comandos a seguir.

### API REST

Para descobrir quais regiões estão disponíveis para um tipo de recurso específico em sua assinatura, use a operação [Listar todos os fornecedores de recursos](https://msdn.microsoft.com/library/azure/dn790524.aspx).

### PowerShell

O exemplo a seguir mostra como obter as regiões com suporte para sites da web.

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations
    
A saída será semelhante a:

    Brazil South
    East Asia
    East US
    Japan East
    Japan West
    North Central US
    North Europe
    South Central US
    West Europe
    West US
    Southeast Asia
    Central US
    East US 2

### CLI do Azure

O exemplo a seguir retorna todos os locais com suporte para cada tipo de recurso.

    azure location list

Também é possível filtrar os resultados de local com um utilitário de JSON como o [jq](https://stedolan.github.io/jq/).

    azure location list --json | jq '.[] | select(.name == "Microsoft.Web/sites")'

Que retorna:

    {
      "name": "Microsoft.Web/sites",
      "location": "Brazil South,East Asia,East US,Japan East,Japan West,North Central US,
            North Europe,South Central US,West Europe,West US,Southeast Asia,Central US,East US 2"
    }

## Versões de API com suporte

Ao implantar um modelo, você deve especificar uma versão de API a ser usada para criar cada recurso. A versão disponível da API corresponde a uma versão das operações da API REST lançadas pelo provedor de recursos. Como um provedor de recursos habilita novos recursos, ele lançará uma nova versão da API REST. Portanto, a versão da API especificada em seu modelo afetará quais propriedades estarão disponíveis para você durante a criação do modelo. Em geral, você deve selecionar a versão mais recente da API ao criar novos modelos. Para os modelos existentes, você pode decidir se deseja continuar usando uma versão de API anterior ou atualizar o modelo para a versão mais recente para aproveitar os novos recursos.

### API REST

Para descobrir quais versões de API está disponíveis para os tipos de recurso, use a operação [Listar todos os provedores de recursos](https://msdn.microsoft.com/library/azure/dn790524.aspx).

### PowerShell

O exemplo a seguir mostra como obter as versões de API disponíveis para um tipo de recurso determinado.

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).ApiVersions
    
A saída será semelhante a:
    
    2015-08-01
    2015-07-01
    2015-06-01
    2015-05-01
    2015-04-01
    2015-02-01
    2014-11-01
    2014-06-01
    2014-04-01-preview
    2014-04-01

### CLI do Azure

Você pode salvar as informações (incluindo as versões disponíveis de API) para um provedor de recursos em um arquivo com o comando a seguir.

    azure provider show Microsoft.Web -vv --json > c:\temp.json

Você pode abrir o arquivo e localizar o elemento **apiVersions**

## Próximas etapas

- Para saber mais sobre a criação de modelos do Gerenciador de Recursos, confira [Criando modelos do Gerenciador de Recursos do Azure](resource-group-authoring-templates.md).
- Para saber mais sobre como implantar recursos, confira [Implantar um aplicativo com o modelo do Gerenciador de Recursos do Azure](resource-group-template-deploy.md).

<!---HONumber=AcomDC_0413_2016-->