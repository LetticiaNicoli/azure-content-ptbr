<properties
	pageTitle="Lista de Tarefas de Desenvolvimento e Administração nos Serviços BizTalk | Microsoft Azure" 
	description="Planejamento e trabalho de auxiliam na implantação dos Serviços BizTalk do Azure."
	services="biztalk-services"
	documentationCenter=""
	authors="msftman"
	manager="erikre"
	editor=""/>

<tags
	ms.service="biztalk-services"
	ms.workload="integration"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/29/2016"
	ms.author="deonhe"/>

# Lista de Tarefas de Desenvolvimento e Administração nos Serviços BizTalk  

## Introdução
Ao trabalhar com Serviços BizTalk do Microsoft Azure, há vários componentes locais e baseados em nuvem a serem considerados. Para começar, considere o seguinte fluxo de processo:

|Etapa|Quem é responsável|Tarefa|Links relacionados|
|----|----|----|----|
1\.|Administrador|Criar a assinatura do Microsoft Azure usando uma conta da Microsoft ou uma conta institucional|[Portal clássico do Azure](http://go.microsoft.com/fwlink/p/?LinkID=213885)|
|2\.|Administrador|Criar ou provisionar um Serviço BizTalk.|[Criar um Serviço BizTalk usando o Portal de clássico do Azure](http://go.microsoft.com/fwlink/p/?LinkID=302280)|
|3\.|Administrador|Registrar você ou a implantação dos Serviços BizTalk da sua empresa|[Registro e Atualização de uma Implantação de Serviço BizTalk no portal dos Serviços BizTalk.](https://msdn.microsoft.com/library/azure/hh689837.aspx)|
|4\.|Administrador|Aplica-se se o aplicativo usa o Serviço de Adaptador do BizTalk para se conectar a um sistema de linha de negócios (LOB) local ou usa uma fila ou destino de tópico. Criar o Namespace do Barramento de Serviço do Azure Atribua valores a esse namespace, ao Nome do Emissor do Barramento de Serviço e à Chave do Emissor do Barramento de Serviço para o desenvolvedor.|[Como Criar ou Modificar um Namespace do Barramento de Serviço](../service-bus/service-bus-dotnet-how-to-use-queues.md) e [Como obter os valores de Nome do Emissor e Chave do Emissor](biztalk-issuer-name-issuer-key.md)|
|5\.|Desenvolvedores|Instalar o SDK e criar o projeto do Serviço BizTalk no Visual Studio.|[Instalar o SDK dos Serviços BizTalk do Azure](https://msdn.microsoft.com/library/azure/hh689760.aspx) e [Criar Pontos de Extremidade de Mensagens Avançadas no Azure](https://msdn.microsoft.com/library/azure/hh689766.aspx)|
|6\.|Desenvolvedores|Implante o projeto de Serviço BizTalk no seu Serviço BizTalk hospedado no Azure.|[Implantação e Atualização do Projeto de Serviços BizTalk](https://msdn.microsoft.com/library/azure/hh689881.aspx)|
|7\.|Administrador|Aplica-se se estiver usando o Eclipse: Você pode adicionar Parceiros e criar Acordos no Portal de Serviços BizTalk do Microsoft Azure. Quando você cria um Contrato, pode adicionar a ponte e/ou Transformações criadas pelo desenvolvedor às Configurações do Contrato.|[Configurando EDI, AS2 e EDIFACT no Portal de Serviços BizTalk](https://msdn.microsoft.com/library/azure/hh689853.aspx)|
|8\.|Administrador|Usando o Portal clássico do Azure, monitore a integridade do seu Serviço BizTalk, incluindo as métricas de desempenho.|[Serviços BizTalk: guias Painel, Monitor e Escala](http://go.microsoft.com/fwlink/p/?LinkID=302281)|
|9\.|Administrador|Usando o Portal dos Serviços BizTalk do Microsoft Azure, gerencie os artefatos usados pelos Serviços BizTalk e acompanhe as mensagens enquanto são processadas pelos arquivos da ponte.|[Usando o Portal dos Serviços BizTalk](https://msdn.microsoft.com/library/azure/dn874043.aspx)|
|10\.|Administrador|Crie um plano de backup para fazer backup do Serviço BizTalk.|[Continuidade de Negócios e Recuperação de Desastres nos Serviços BizTalk](https://msdn.microsoft.com/library/azure/dn509557.aspx) |  
## Próximas etapas
[Tutoriais e Amostras](https://msdn.microsoft.com/library/azure/hh689895.aspx)

[Criar um projeto no Visual Studio](https://msdn.microsoft.com/library/azure/hh689811.aspx)

[Instalar o SDK dos Serviços BizTalk do Azure](https://msdn.microsoft.com/library/azure/hh689760.aspx)

## Conceitos
[Criar o projeto no Visual Studio](https://msdn.microsoft.com/library/azure/hh689811.aspx) [Sistemas de Mensagens EDI, AS2 e EDIFACT (entre empresas)](https://msdn.microsoft.com/library/azure/hh689898.aspx)
## Outros recursos  
[Adicionar Origem, Destino e Pontos de Extremidade de Mensagens](https://msdn.microsoft.com/library/azure/hh689877.aspx) [Aprender e criar Mapas de Mensagem e Transformações](https://msdn.microsoft.com/library/azure/hh689905.aspx) [Usando o Serviço de Adaptador do BizTalk (BAS)](https://msdn.microsoft.com/library/azure/hh689889.aspx) [Serviços BizTalk do Azure](http://go.microsoft.com/fwlink/p/?LinkID=303664)

<!---HONumber=AcomDC_0420_2016-->