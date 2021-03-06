<properties
	pageTitle="Movendo dados para dentro e para fora do Armazenamento do Azure | Microsoft Azure"
	description="Este artigo fornece uma visão geral dos diferentes métodos de mover dados para dentro e para fora do Armazenamento do Azure."
	services="storage"
	documentationCenter=""
	authors="micurd"
	manager="jahogg"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/18/2016"
	ms.author="micurd"/>

# Movendo dados para dentro e para fora do Armazenamento do Azure

Se você quiser mover dados locais para o Armazenamento do Azure (ou vice-versa), há várias maneiras de fazer isso. A melhor abordagem para você dependerá do seu cenário. Este artigo fornece uma visão geral rápida de diferentes cenários e ofertas apropriadas para cada um.

## Criando aplicativos

Se você estiver criando um aplicativo, o desenvolvimento na API REST ou em uma das muitas de nossas bibliotecas de cliente é uma excelente maneira de mover dados para dentro e para fora do Armazenamento do Azure.

O Armazenamento do Azure fornece ricas bibliotecas de cliente para .NET, iOS, Java, Android, UWP (Plataforma Universal do Windows), Xamarin, C++, Node.JS, PHP, Ruby e Python. As bibliotecas de cliente oferecem recursos avançados, como lógica de recuperação, registro em log e carregamentos paralelos. Você também pode desenvolver diretamente na API REST, que pode ser chamada por qualquer linguagem que faça solicitações HTTP/HTTPS.

Confira [Introdução ao Armazenamento de Blobs do Azure](storage-dotnet-how-to-use-blobs.md) para saber mais.

## Exibindo/interagindo rapidamente com seus dados

Se você quiser uma maneira fácil de exibir os dados do Armazenamento do Azure e, ao mesmo, a capacidade de carregá-los e baixá-los, considere usar um Gerenciador de Armazenamento do Azure.

Confira nossa lista de [Gerenciadores de Armazenamento do Azure](storage-explorers.md) para saber mais.

## Administração do Sistema

Se você se sentir mais à vontade com um utilitário de linha de comando (por exemplo, Administradores de Sistema), veja algumas das opções a serem consideradas:

### AzCopy

O AzCopy é um utilitário de linha de comando do Windows desenvolvido para cópia de dados de alto desempenho para dentro e para fora do Armazenamento do Azure. Você também pode copiar dados em uma conta de armazenamento ou entre diferentes contas de armazenamento.

Confira [Transferir dados com o utilitário de linha de comando AzCopy](storage-use-azcopy.md) para saber mais.

### PowerShell do Azure

O Azure PowerShell é um módulo que fornece cmdlets para gerenciar serviços no Azure. Trata-se de uma linguagem de scripts e shell de linha de comando baseada em tarefas projetada especialmente para administração do sistema.

Confira [Usando o Azure PowerShell com o Armazenamento do Azure](storage-powershell-guide-full.md) para saber mais.

### CLI do Azure

A CLI do Azure fornece um conjunto de comandos entre plataformas de software livre para trabalhar com os serviços do Azure. A CLI do Azure está disponível no Windows, OSX e Linux.

Confira [Usando a CLI do Azure com o Armazenamento do Azure](storage-azure-cli.md) para saber mais.

## Mover grandes quantidades de dados com uma rede lenta

Um dos maiores desafios associados à movimentação de grandes quantidades de dados é o tempo de transferência. Se você deseja introduzir dados no Armazenamento do Azure ou extrair dados dele sem se preocupar com os custos de rede ou em escrever código, a Importação/Exportação do Azure é a solução adequada.

Confira [Importação/exportação do Azure](storage-import-export-service.md) para saber mais.

## Fazendo backup dos dados

Se você simplesmente precisa fazer backup dos dados no Armazenamento do Azure, o Backup do Azure é a melhor opção. Trata-se de uma solução potente para fazer backup de dados locais e de VMs do Azure.

Confira [Backup do Azure](../backup/backup-introduction-to-azure-backup.md) para saber mais.

## Acessando dados locais e na nuvem

Se você precisar de uma solução para acessar seus dados locais e na nuvem, considere usar a solução de armazenamento de nuvem híbrida do Azure, StorSimple. Essa solução consiste em um dispositivo StorSimple físico que, de maneira inteligente, armazena dados usados com frequência em SSDs, dados usados ocasionalmente em HDDs e dados inativos/de backup/de arquivamento no Armazenamento do Azure.

Confira [StorSimple](../storsimple/storsimple-overview.md) para saber mais.

## Recuperando dados

Quando houver cargas de trabalho e aplicativos locais, você precisará de uma solução que permita aos negócios continuar em execução no caso de um desastre. O Azure Site Recovery cuida da replicação, do failover e da recuperação de máquinas virtuais e servidores físicos. Os dados replicados são armazenados no Armazenamento do Azure, permitindo que você elimine a necessidade de um datacenter local secundário.

Confira [Azure Site Recovery](../site-recovery/site-recovery-overview.md) para saber mais.

<!---HONumber=AcomDC_0323_2016-->