<properties
	pageTitle="Configurações personalizadas para Ambientes de Serviço de Aplicativo"
	description="Definições de configuração personalizadas para Ambientes de Serviço de Aplicativo"
	services="app-service"
	documentationCenter=""
	authors="stefsch"
	manager="nirma"
	editor=""/>

<tags
	ms.service="app-service"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/08/2016"
	ms.author="stefsch"/>

# Definições de configuração personalizadas para Ambientes de Serviço de Aplicativo

## Visão geral ##
Como os Ambientes de Serviço de Aplicativo são isolados em um único cliente, há certas definições de configuração que podem ser aplicadas exclusivamente a Ambientes de Serviço de Aplicativo. Este artigo documenta as várias personalizações específicas que estão disponíveis para Ambientes de Serviço de Aplicativo.

Você pode armazenar as personalizações de Ambiente de Serviço de Aplicativo usando uma matriz no novo atributo **clusterSettings**. Esse atributo é encontrado no dicionário de "Propriedades" da entidade do Azure Resource Manager *hostingEnvironments*.

O trecho de código de modelo do Resource Manager abreviado a seguir mostra o atributo **clusterSettings**:


    "resources": [
    {
       "apiVersion": "2015-08-01",
       "type": "Microsoft.Web/hostingEnvironments",
       "name": ...,
       "location": ...,
       "properties": {
          "clusterSettings": [
             {
                 "name": "nameOfCustomSetting",
                 "value": "valueOfCustomSetting"
             }
          ],
          "workerPools": [ ...],
          etc...
       }
    }

O atributo **clusterSettings** pode ser incluído em um modelo do Resource Manager para atualizar o Ambiente de Serviço de Aplicativo.

## Usar o Gerenciador de Recursos do Azure para atualizar um Ambiente de Serviço de Aplicativo
Como alternativa, você pode atualizar o Ambiente de Serviço de Aplicativo usando o [Gerenciador de Recursos do Azure](https://resources.azure.com).

1. No Gerenciador de Recursos, acesse o nó para Ambiente de Serviço de Aplicativo (**assinaturas** > **resourceGroups** > **provedores** > **Micrososft.Web** > **hostingEnvironments**). Em seguida, clique no Ambiente de Serviço de Aplicativo específico que você deseja atualizar.

2. No painel à direita, clique em **Leitura/gravação** na barra de ferramentas superior para permitir a edição interativa no Gerenciador de Recursos.

3. Clique no botão azul **Editar** para tornar o modelo do Resource Manager editável.

4. Role até o final do painel à direita. O atributo **clusterSettings** está na parte inferior, na qual você poderá inserir ou atualizar seu valor.

5. Digite (ou copie e cole) a matriz de valores de configuração desejada no atributo **clusterSettings**.

6. Clique no botão verde **PUT** localizado na parte superior do painel à direita para confirmar a alteração no Ambiente de Serviço de Aplicativo.

No entanto, você envia a alteração, isso demora aproximadamente 30 minutos, multiplicados pelo número de front-ends no Ambiente de Serviço de Aplicativo, para que a alteração tenha efeito. Por exemplo, se um Ambiente de Serviço de Aplicativo tiver quatro front-ends, levará aproximadamente duas horas para que a atualização de configuração seja concluída. Embora a alteração de configuração esteja sendo revertida, nenhuma outra operação de colocação em escala ou operação de alteração pode ocorrer no Ambiente de Serviço de Aplicativo.

## Desabilitar o TLS 1.0 ##
Uma dúvida recorrente dos clientes, principalmente daqueles lidando com auditorias de conformidade de PCI, é como desabilitar explicitamente o TLS 1.0 para seus aplicativos.

O TLS 1.0 pode ser desabilitado por meio da seguinte entrada de **clusterSettings**:

        "clusterSettings": [
            {
                "name": "DisableTls1.0",
                "value": "1"
            }
        ],



## Introdução
O site de modelo do Azure Quickstart Resource Manager inclui um modelo com a definição básica para a [criação de um Ambiente de Serviço de Aplicativo](https://azure.microsoft.com/documentation/templates/201-web-app-ase-create/).


<!-- LINKS -->

<!-- IMAGES -->

<!---HONumber=AcomDC_0420_2016-->