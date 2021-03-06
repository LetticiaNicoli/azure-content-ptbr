<properties
   pageTitle="Recuperação de desastre do Azure Service Fabric | Microsoft Azure"
   description="O Azure Service Fabric oferece os recursos necessários para lidar com todos os tipos de desastres. Este artigo descreve os tipos de desastres que podem ocorrer e como lidar com eles."
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="03/03/2016"
   ms.author="seanmck"/>

# Recuperação de desastre no Azure Service Fabric

Uma parte essencial do fornecimento de um aplicativo em nuvem de alta disponibilidade é garantir que ele possa sobreviver a todos os diferentes tipos de falhas, incluindo aquelas que estão completamente fora do seu controle. Este artigo descreve o layout físico de um cluster do Azure Service Fabric no contexto de desastres potenciais e oferece orientação sobre como lidar com tais desastres para limitar ou eliminar o risco de tempo de inatividade ou de perda de dados.

## Layout físico dos clusters do Service Fabric no Azure

Para entender o risco representado por diferentes tipos de falhas, é útil saber como clusters estão fisicamente dispostos no Azure.

Quando você cria um cluster do Service Fabric no Azure, precisa escolher uma região onde ele ficará hospedado. A infraestrutura do Azure configura os recursos para esse cluster dentro da região, principalmente o número de VMs (máquinas virtuais) solicitado. Vamos examinar mais de perto como e onde as VMs são provisionadas.

### Domínios de falha

Por padrão, as VMs do cluster serão distribuídas uniformemente entre grupos lógicos conhecidos como FDs (domínios de falha), que segmentam as máquinas com base em possíveis falhas no hardware do host. Mais especificamente, se duas VMs residirem em dois FDs diferentes, saiba que elas não compartilharão a mesma fonte de energia ou o mesmo comutador de rede. Como resultado, uma falha na rede local ou uma falha de energia que afete uma VM não afetará a outra, permitindo que o Service Fabric balanceie novamente a carga de trabalho da máquina que não responde no cluster.

Você pode visualizar o layout do seu cluster em domínios de falha usando o mapa de cluster fornecido no [Gerenciador do Service Fabric](service-fabric-visualizing-your-cluster.md):

![Nós espalhados pelos domínios de falha no Service Fabric Explorer][sfx-cluster-map]

>[AZURE.NOTE] O outro eixo no mapa de cluster mostra os domínios de atualização, que logicamente agrupam nós com base nas atividades de manutenção planejadas. Os clusters do Service Fabric no Azure sempre ficam dispostos em cinco domínios de atualização.

### Distribuição geográfica

No momento, existem 22 regiões do Azure no mundo, com mais cinco já anunciadas. Uma região individual pode conter um ou mais data centers físicos, dependendo da demanda e da disponibilidade dos locais adequados, entre outros fatores. No entanto, observe que mesmo em regiões com vários data centers físicos, não há nenhuma garantia de que as VMs do cluster serão distribuídas uniformemente entre os locais físicos. Na verdade, no momento, todas as VMs de um determinado cluster são provisionadas em um único local físico.

## Lidando com falhas

Há vários tipos de falhas que podem afetar seu cluster, cada uma com sua própria atenuação. Vamos examiná-las na ordem de probabilidade de ocorrência.

### Falhas individuais de computador

Como mencionado, as falhas individuais de computadores, dentro da VM ou no hardware ou no software que a hospedam em um domínio de falha, não apresentam risco. O Service Fabric normalmente detectará a falha em segundos e responderá adequadamente com base no estado do cluster. Por exemplo, se o nó estivesse hospedando as réplicas primárias de uma partição, um novo primário será eleito a partir das réplicas secundárias da partição. Quando o Azure restaura a máquina que falhou, a faz reingressar no cluster de forma automática e ela assume novamente sua cota da carga de trabalho.

### Várias falhas simultâneas de máquina

Enquanto os domínios de falha reduzem significativamente o risco de falhas simultâneas da máquina, sempre há a possibilidade de várias falhas aleatórias tornarem vários computadores inoperantes simultaneamente em um cluster.

Em geral, contanto que a maioria dos nós permaneça disponível, o cluster continuará a operar, embora com menor capacidade, já que as réplicas com estado são empacotadas em um conjunto menor de máquinas e menos instâncias sem estado ficam disponíveis para distribuir a carga.

#### Perda de quorum

Se a maioria das réplicas de partição de um serviço com estado ficar inoperante, essa partição entrará em um estado conhecido como "perda de quorum". Nessa hora, o Service Fabric irá parar, permitindo gravações nessa partição para garantir que o estado permaneça consistente e confiável. Na verdade, estamos optando por aceitar um período de indisponibilidade para garantir que os clientes não sejam informados de que seus dados foram salvos quando na verdade não foram. Observe que, se você tiver aceitado permitir leituras das réplicas secundárias para o serviço com estado, poderá continuar a executar as operações de leitura nesse estado. Uma partição permanecerá na perda de quorum até que um número suficiente de réplicas volte ou até que o administrador do cluster force a movimentação do sistema usando a [API Repair-ServiceFabricPartition](repair-partition-ps). A execução dessa ação quando a réplica primária estiver inoperante resultará em perda de dados.

Os serviços do sistema também podem sofrer perda de quorum, e o impacto será específico para o serviço em questão. Por exemplo, a perda de quorum no serviço de nomenclatura afetará a resolução de nomes, ao passo que a perda de quorum no serviço gerenciador de failover bloqueará os failovers e a criação de novos serviços. Observe que, diferentemente dos seus próprios serviços, a tentativa de reparar os serviços do sistema *não* é recomendada. Em vez disso, é preferível simplesmente aguardar até que as réplicas desativadas voltem.

#### Minimizando o risco de perda de quorum

Você pode minimizar o risco de perda de quorum aumentando o tamanho do conjunto de réplicas de destino para o serviço. É útil considerar o número de réplicas necessárias em termos do número de nós indisponíveis que você pode tolerar ao mesmo tempo enquanto permanece disponível para gravação, tendo em mente que atualizações de cluster ou de aplicativo podem fazer os nós ficarem temporariamente indisponíveis, além de falhas de hardware.

Considere os exemplos a seguir, supondo que você tenha configurado seus serviços para ter um MinReplicaSetSize três, o menor número recomendado para serviços de produção. Com um TargetReplicaSetSize três (um principal e dois secundários), uma falha de hardware durante uma atualização (duas réplicas inoperantes) resultará em perda de quorum e o serviço se tornará somente leitura. Como alternativa, se você tiver cinco réplicas, será capaz de suportar duas falhas durante a atualização (três réplicas inoperantes), já que as duas réplicas restantes ainda podem formar um quorum dentro do conjunto de réplicas mínimo.

### Interrupções ou destruição do data center

Raramente, os data centers físicos podem se tornar temporariamente indisponíveis devido à perda de energia ou de conectividade de rede. Nesses casos, os clusters e os aplicativos do Service Fabric estarão indisponíveis da mesma forma, mas os dados serão preservados. Para os clusters em execução no Azure, você pode exibir as atualizações sobre interrupções na [página de status do Azure](azure-status-dashboard).

Na hipótese altamente improvável de um data center físico inteiro ser destruído, os clusters do Service Fabric hospedados nele serão perdidos, juntamente com seu estado.

Para evitar essa possibilidade, é extremamente importante [fazer backup do seu estado de backup](service-fabric-reliable-services-backup-restore.md) periodicamente em um armazenamento com redundância geográfica e validar a sua capacidade de restaurá-lo. Com que frequência você fará um backup dependerá de seu RPO (objetivo de ponto de recuperação). Mesmo se você não tiver implementado totalmente o backup e a restauração, deverá implementar um manipulador para o evento `OnDataLoss`, para que possa registrar o log quando ele ocorrer da seguinte forma:

```c#
protected virtual Task<bool> OnDataLoss(CancellationToken cancellationToken)
{
  ServiceEventSource.Current.ServiceMessage(this, "OnDataLoss event received.");
  return Task.FromResult(true);
}
```

>[AZURE.NOTE] O backup e a restauração só estão disponíveis para a API Reliable Services. O backup e a restauração para Reliable Actors estarão disponíveis em uma versão futura.

### Falhas de software e outras fontes de perda de dados

Os defeitos de código de serviços, os erros humanos operacionais e as violações de segurança são mais comuns como causa de perda de dados do que as falhas generalizadas de data centers. No entanto, em todos os casos, a estratégia de recuperação é a mesma: faça backups regulares de todos os serviços com estado e exercite a capacidade de restaurar esse estado.

## Próximas etapas

- Saiba como simular várias falhas usando a [estrutura de capacidade de teste](service-fabric-testability-overview.md)
- Leia outros recursos de recuperação de desastres e alta disponibilidade. A Microsoft publicou várias orientações sobre estes tópicos. Embora alguns desses documentos mencionem técnicas específicas para uso em outros produtos, eles contêm várias práticas recomendadas gerais que também se aplicam ao contexto do Service Fabric:
 - [Lista de verificação de disponibilidade](azure-availability-checklist)
 - [Executando a análise de recuperação de desastre](disaster-recovery-drill)
 - [Recuperação de desastre e alta disponibilidade para aplicativos do Azure](dr-ha-guide)


<!-- External links -->

[repair-partition-ps]: https://msdn.microsoft.com/pt-BR/library/mt163522.aspx
[azure-status-dashboard]: https://azure.microsoft.com/pt-BR/status/
[azure-availability-checklist]: https://azure.microsoft.com/pt-BR/documentation/articles/best-practices-availability-checklist/
[disaster-recovery-drill]: https://azure.microsoft.com/pt-BR/documentation/articles/sql-database-disaster-recovery-drills/

<!-- Images -->

[sfx-cluster-map]: ./media/service-fabric-disaster-recovery/sfx-clustermap.png

<!---HONumber=AcomDC_0323_2016-->