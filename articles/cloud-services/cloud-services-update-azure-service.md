<properties
pageTitle="Como atualizar um serviço de nuvem | Microsoft Azure"
description="Saiba como atualizar os serviços de nuvem no Azure. Saiba como uma atualização em um serviço de nuvem é realizada para garantir a disponibilidade."
services="cloud-services"
documentationCenter=""
authors="kenazk"
manager="timlt"
editor=""/>
<tags
ms.service="cloud-services"
ms.workload="tbd"
ms.tgt_pltfrm="na"
ms.devlang="na"
ms.topic="article"
ms.date="10/26/2015"
ms.author="kenazk"/>

# Como atualizar um serviço de nuvem

## Visão geral
A 10.000 pés, atualizar um serviço de nuvem, incluindo suas funções e o SO convidado, é um processo que envolve três etapas. Primeiro, os binários e os arquivos de configuração do novo serviço de nuvem, ou a versão do sistema operacional, devem ser carregados. Em seguida, o Azure reserva recursos de computação e de rede para o serviço de nuvem com base nos requisitos da nova versão do serviço de nuvem. Por fim, o Azure executa uma atualização sem interrupção a fim de atualizar de forma incremental o locatário para a nova versão ou o SO convidado, preservando a disponibilidade. Este artigo discute os detalhes desta última etapa – a atualização sem interrupção.

## Atualizar um serviço do Azure

O Azure organiza suas instâncias de função em agrupamentos lógicos chamados de domínios de atualização (UD). Os domínios de atualização (UD) são conjuntos lógicos de instâncias de função que são atualizados como um grupo. O Azure atualiza um serviço de nuvem um UD por vez, o que permite que instâncias em outros UDs continuem a atender ao tráfego.

O número padrão de domínios de atualização é de cinco. Você pode especificar um número diferente de domínios de atualização incluindo o atributo upgradeDomainCount no arquivo de definição do serviço (.csdef). Para saber mais sobre o atributo upgradeDomainCount, consulte [Esquema WebRole](https://msdn.microsoft.com/library/azure/gg557553.aspx) ou [Esquema WorkerRole](https://msdn.microsoft.com/library/azure/gg557552.aspx).

Quando você executa uma atualização in-loco de uma ou mais funções em seu serviço, o Azure atualiza conjuntos de instâncias de função de acordo com o domínio de atualização ao qual elas pertencem. O Azure atualiza todas as instâncias em um determinado domínio de atualização. Ele faz isso interrompendo as atualizações, atualizando-as, colocando-as online novamente e, em seguida, passando-as para o próximo domínio. Ao interromper somente as instâncias em execução no domínio de atualização atual, o Azure garante que uma atualização ocorrerá com o menor impacto possível no serviço em execução. Para saber mais, consulte [Como atualizar os lucros](https://msdn.microsoft.com/library/azure/Hh472157.aspx#proceed) posteriormente neste artigo.

> [AZURE.NOTE] Embora os termos **atualização** e **upgrade** tenham uma pequena diferença dentro do contexto do Azure, eles podem ser usados de forma intercambiável para os processos e descrições dos recursos neste documento.

O serviço deve definir pelo menos duas instâncias de uma função para que essa função seja atualizada in-loco sem tempo de inatividade. Se o serviço for composto por apenas uma instância de uma função, seu serviço ficará indisponível até a conclusão da atualização in-loco.

Este tópico aborda as seguintes informações sobre as atualizações do Azure:

-   [Alterações permitidas no serviço durante uma atualização](#AllowedChanges)
-   [Reversão de uma atualização](#RollbackofanUpdate)
-   [Inicialização de várias operações de mutação em uma implantação em andamento](#multiplemutatingoperations)
-   [Distribuição de funções em domínios de atualização](#distributiondfroles)
-   [Como uma atualização é realizada](#howanupgradeproceeds)

## Alterações permitidas no serviço durante uma atualização
A tabela a seguir mostra as alterações permitidas em um serviço durante uma atualização:

|Alterações permitidas na hospedagem, serviços e funções|Atualização in-loco|Em estágios (permuta de VIP)|Excluir e reimplantar|
|---|---|---|---|
|Versão do sistema operacional|Sim|Sim|Sim
|Nível de confiança do .NET|Sim|Sim|Sim|
|Tamanho da máquinas virtual|Sim*|Sim|Sim|
|Configurações de armazenamento local|Apenas aumento|Sim|Sim|
|Adicionar ou remover funções em um serviço|Sim|Sim|Sim|
|Número de instâncias de uma função específica|Sim|Sim|Sim|
|Número ou tipo de pontos de extremidade de um serviço|Sim*|Não|Sim|
|Nomes e valores dos parâmetros de configuração|Sim|Sim|Sim|
|Valores (mas não nomes) dos parâmetros de configuração|Sim|Sim|Sim|
|Adicionar novos certificados|Sim|Sim|Sim|
|Alterar certificados existentes|Sim|Sim|Sim|
|Implantar novo código|Sim|Sim|Sim|
*Exige o Azure SDK 1.5 ou versões posteriores.

> [AZURE.WARNING] A alteração do tamanho da máquina virtual destruirá os dados locais.


Os itens a seguir não têm suporte durante uma atualização:

-   Alteração do nome de uma função. Remova e, em seguida, adicione a função com o novo nome.
-   Alteração da contagem de Domínio de atualização.
-   Redução do tamanho dos recursos locais.

Se você estiver fazendo outras atualizações na definição do serviço, como a redução do tamanho do recurso local, será necessário executar uma atualização de permuta de VIP. Para saber mais, consulte, consulte [Implantação de permuta](https://msdn.microsoft.com/library/azure/ee460814.aspx).

## Como uma atualização é realizada
Você pode decidir se deseja atualizar todas as funções ou uma única função no serviço. Em ambos os casos, todas as instâncias de cada função que está sendo atualizada e pertence ao primeiro domínio de atualização serão interrompidas, atualizadas e colocadas novamente online. Quando estiverem online novamente, as instâncias no segundo domínio de atualização serão interrompidas, atualizadas e colocadas novamente online. Um serviço de nuvem pode ter no máximo uma atualização ativa por vez. A atualização é sempre executada na versão mais recente do serviço de nuvem.

O diagrama a seguir ilustra como a atualização ocorre se você estiver atualizando todas as funções no serviço:

![Atualizar serviço](media/cloud-services-update-azure-service/IC345879.png "Atualizar serviço")

Este próximo diagrama ilustra como a atualização ocorre se você estiver atualizando somente uma única função:

![Atualizar função](media/cloud-services-update-azure-service/IC345880.png "Atualizar função")

> [AZURE.NOTE] Ao atualizar um serviço de uma única instância para várias instâncias, seu serviço será interrompido enquanto a atualização estiver em execução, devido ao modo como o Azure atualiza os serviços. A garantia de disponibilidade do serviço do contrato de nível de serviço só se aplica aos serviços implantados com mais de uma instância. A lista a seguir descreve como os dados em cada unidade são afetados por cada cenário de atualização de serviço do Azure:
>
>Reinicialização da VM:
>
-   C: preservado
-   D: preservados
-   E: preservados
>
>Reinicialização do portal:
>
-   C: preservado
-   D: preservados
-   E: destruídos
>
>Nova imagem do Portal:
>
-   C: preservado
-   D: destruídos
-   E: destruídos

>Atualização in-loco:
>
-   C: preservado
-   D: preservados
-   E: destruídos
>
>Migração de nó:
>
-   C: destruídos
-   D: destruídos
-   E: destruídos

>Observe que, na lista acima, a unidade E: representa a unidade raiz da função e não deve ser codificada. Em vez disso, use a variável de ambiente % RoleRoot % para representar a unidade.

>Para minimizar o tempo de inatividade ao atualizar um serviço de instância única, implante um novo serviço de várias instâncias no servidor de preparo e execute uma permuta de VIP.

Durante uma atualização automática, o Azure Fabric Controller avalia periodicamente a integridade do serviço de nuvem a fim de determinar quando é seguro passar ao próximo UD. Essa avaliação de integridade é executada de acordo com a função e considera apenas instâncias na versão mais recente (ou seja, instâncias de UDs que já foram descritos). Ela verifica se um número mínimo de instâncias de função, para cada função, atingiu um estado de terminal satisfatório.

### Tempo limite de inicialização de instância de função
O controlador de malha aguardará 30 minutos para cada instância de função atingir um estado iniciado. Se a duração do tempo limite expirar, o controlador de malha prosseguirá para a próxima instância de função.

## Reversão de uma atualização
O Azure fornece flexibilidade no gerenciamento de serviços durante uma atualização, permitindo que você inicie outras operações em um serviço, após a aceitação da solicitação de atualização inicial pelo controlador de malha do Azure. Uma reversão só pode ser realizada quando uma atualização (mudança de configuração) ou upgrade estiver no estado **em andamento** na implantação. Uma atualização ou upgrade é considerada em andamento desde que exista pelo menos uma instância do serviço que ainda não foi atualizada para a nova versão. Para testar se há permissão para uma reversão, verifique se o valor do sinalizador RollbackAllowed, retornado pelas operações [Obter Implantação](https://msdn.microsoft.com/library/azure/ee460804.aspx) e [Obter Propriedades do Serviço de Nuvem](https://msdn.microsoft.com/library/azure/ee460806.aspx), está definido como true.

> [AZURE.NOTE] Só faz sentido chamar a Reversão em uma atualização ou upgrade **in-loco** porque os upgrades de permuta de VIP envolvem a substituição de toda uma instância em execução do seu serviço por outra.

A Reversão de uma atualização em andamento tem os seguintes efeitos sobre a implantação:

-   Quaisquer instâncias de função que ainda não foram atualizadas para a nova versão não serão atualizadas, porque essas instâncias já estão executando a versão de destino do serviço.
-   Quaisquer instâncias de função que já foram atualizadas para a nova versão do arquivo de pacote (\*.cspkg) do serviço ou do arquivo de configuração (\*.cscfg) do serviço (ou ambos os arquivos) serão revertidas para a versão pré-atualização desses arquivos.

Essa funcionalidade é fornecida pelos seguintes recursos:

-   A operação [Reversão de Atualização ou Upgrade](https://msdn.microsoft.com/library/azure/hh403977.aspx), que pode ser chamada em uma atualização de configuração (disparada chamando [Alterar configuração de implantação](https://msdn.microsoft.com/library/azure/ee460809.aspx)) ou upgrade (disparado chamando [Implantação de atualização](https://msdn.microsoft.com/library/azure/ee460793.aspx)) desde que haja pelo menos uma instância do serviço que ainda não tenha sido atualizada para a nova versão.
-   O elemento Bloqueado e o elemento RollbackAllowed, que retornam como parte do corpo da resposta das operações [Obter a implantação](https://msdn.microsoft.com/library/azure/ee460804.aspx) e [Obter propriedades do serviço de nuvem](https://msdn.microsoft.com/library/azure/ee460806.aspx):
    1.  O elemento Locked permite a você detectar quando uma operação de mutação pode ser chamada em uma determinada implantação.
    2.  O elemento RollbackAllowed permite detectar quando a operação [Reversão de atualização ou upgrade](https://msdn.microsoft.com/library/azure/hh403977.aspx) pode ser chamada em uma determinada implantação.

    Para executar uma reversão, você não precisa verificar os elementos Locked e RollbackAllowed. Basta confirmar se RollbackAllowed está definido como true. Esses elementos retornam apenas se esses métodos forem chamados usando o cabeçalho de solicitação definido como "x-ms-version: 2011-10-01" ou uma versão posterior. Para saber mais sobre os cabeçalhos de controle de versão, consulte [Controle de versão do serviço de gerenciamento](https://msdn.microsoft.com/library/azure/gg592580.aspx).

Há algumas situações nas quais não há suporte para uma reversão de uma atualização ou upgrade, entre elas:

-   Redução nos recursos locais - se a atualização aumentar os recursos locais para uma função, a plataforma do Azure não permitirá a reversão. Para saber mais sobre como configurar os recursos locais para uma função, consulte [Configurar os recursos de armazenamento local](https://msdn.microsoft.com/library/azure/ee758708.aspx).
-   Limites de cota - se a atualização foi uma operação de redução vertical, talvez você não tenha cota de computação suficiente para concluir a operação de reversão. Cada assinatura do Azure tem uma cota associada que especifica o número máximo de núcleos que podem ser consumidos por todos os serviços hospedados que pertencem a essa assinatura. Se a execução de uma reversão de uma determinada atualização colocar sua assinatura acima da cota, a reversão não será habilitada.
-   Condição de corrida - Se a atualização inicial tiver sido concluída, não será possível realizar uma reversão.

Um exemplo de quando a reversão de uma atualização pode ser útil é se você estiver usando a operação [Implantação de atualização](https://msdn.microsoft.com/library/azure/ee460793.aspx) no modo manual para controlar a taxa com a qual uma grande atualização in-loco em seu serviço hospedado do Azure é implantada.

Durante a implantação da atualização você chama a [Implantação de atualização](https://msdn.microsoft.com/library/azure/ee460793.aspx) no modo manual e começa a percorrer os domínios de atualização. Se, em algum momento, enquanto monitora a atualização, você observar algumas instâncias de função nos primeiros domínios de atualização ficarem sem resposta, chame a operação [Reversão de atualização ou upgrade](https://msdn.microsoft.com/library/azure/hh403977.aspx) na implantação, e isso deixará as instâncias que ainda não tinham sido atualizadas sem alteração e reverterá as instâncias que foram atualizadas para o pacote de serviço e configuração anteriores.

## Inicialização de várias operações de mutação em uma implantação em andamento
Em alguns casos, convém iniciar várias operações de mutação simultâneas em uma implantação em andamento. Por exemplo, você pode executar uma atualização de serviço e, enquanto a atualização estiver sendo revertida em seu serviço, convém fazer algumas alterações por exemplo, reverter a atualização, aplicar outra atualização ou até mesmo excluir a implantação. Um caso em que isso pode ser necessário é se uma atualização de serviço contiver um código com bugs que faz com que uma instância de função atualizada apresente falhas repetidamente. Nesse caso, o controlador de malha do Azure não poderá progredir com a aplicação dessa atualização, pois há um número insuficiente de instâncias íntegras no domínio atualizado. Esse estado é conhecido como *implantação paralisada*. Você pode resolver a paralisação da implantação revertendo a atualização ou aplicando uma nova atualização sobre a atualização com falha.

Após o recebimento da solicitação inicial de atualização ou upgrade pelo controlador de malha do Azure, você pode iniciar operações de mutação subsequentes. Ou seja, não é necessário esperar que a operação inicial seja concluída antes de iniciar outra operação de mutação.

Iniciar uma segunda operação de atualização enquanto a primeira atualização está em andamento terá um desempenho semelhante à operação de reversão. Se a segunda atualização estiver em modo automático, o primeiro domínio de atualização será atualizado imediatamente, possivelmente fazendo com que vários domínios de atualização fiquem offline no mesmo período.

As operações de mutação são as seguintes: [Alterar configuração de implantação](https://msdn.microsoft.com/library/azure/ee460809.aspx), [Implantação de atualização](https://msdn.microsoft.com/library/azure/ee460793.aspx), [Status da implantação de atualização](https://msdn.microsoft.com/library/azure/ee460808.aspx), [Excluir implantação](https://msdn.microsoft.com/library/azure/ee460815.aspx) e [Reversão de atualização ou upgrade](https://msdn.microsoft.com/library/azure/hh403977.aspx).

Duas operações, [Obter a implantação](https://msdn.microsoft.com/library/azure/ee460804.aspx) e [Obter propriedades do serviço de nuvem](https://msdn.microsoft.com/library/azure/ee460806.aspx), retornam o sinalizador Locked, que pode ser examinado a fim de determinar se uma operação de mutação pode ser chamada em uma determinada implantação.

Para chamar a versão desses métodos que retorna o sinalizador Locked, você deve definir o cabeçalho de solicitação como "x-ms-version: 2011-10-01" ou posterior. Para saber mais sobre os cabeçalhos de controle de versão, consulte [Controle de versão do serviço de gerenciamento](https://msdn.microsoft.com/library/azure/gg592580.aspx).

## Distribuição de funções em domínios de atualização
O Azure distribui instâncias de uma função uniformemente por um determinado número de domínios de atualização, que pode ser configurado como parte do arquivo de definição (.csdef) do serviço. O número máximo de domínios de atualização é de 20 e o padrão é cinco. Para saber mais sobre como modificar o arquivo de definição de serviço, consulte [Esquema de definição de serviço do Azure (arquivo .csdef)](https://msdn.microsoft.com/library/azure/ee758711.aspx).

Por exemplo, se a sua função tiver dez instâncias, por padrão cada domínio de atualização conterá duas instâncias. Se a sua função tiver 14 instâncias, quatro dos domínios de atualização conterão três instâncias e um quinto domínio conterá duas.

Os domínios de atualização são identificados com um índice baseado em zero: o primeiro domínio de atualização tem uma ID de 0 e o segundo domínio de atualização tem uma ID de 1 e assim por diante.

O diagrama a seguir ilustra como é a distribuição de um serviço que contém duas funções quando o serviço define dois domínios de atualização. O serviço está executando oito instâncias da função Web e nove instâncias da função de trabalho.

![Distribuição de domínios de atualização](media/cloud-services-update-azure-service/IC345533.png "Distribuição de domínios de atualização")

> [AZURE.NOTE] Observe que o Azure controla como as instâncias são alocadas nos domínios de atualização. Não é possível especificar quais instâncias são alocadas para cada domínio.

## Próximas etapas
[Como gerenciar os serviços de nuvem](cloud-services-how-to-manage.md)<br> [Como monitorar os serviços de nuvem](cloud-services-how-to-monitor.md)<br> [Como configurar os serviços de nuvem](cloud-services-how-to-configure.md)<br>

<!---HONumber=AcomDC_0413_2016-->