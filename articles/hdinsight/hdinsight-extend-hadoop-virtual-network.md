<properties
	pageTitle="Estender o HDInsight com a Rede Virtual | Microsoft Azure"  
	description="Saiba como usar a Rede Virtual do Azure para conectar o HDInsight a outros recursos de nuvem ou recursos no seu datacenter"
	services="hdinsight"
	documentationCenter=""
	authors="Blackmist"
	manager="paulettm"
	editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="05/04/2016"
   ms.author="larryfr"/>


#Estender os recursos do HDInsight usando a Rede Virtual do Azure

A Rede Virtual do Azure permite que você estenda suas soluções Hadoop para incorporar recursos locais, como SQL Server, ou para criar redes privadas seguras entre recursos na nuvem.

[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell-and-cli.md)]


##<a id="whatis"></a>O que é a Rede Virtual do Azure?

A [Rede Virtual do Azure](https://azure.microsoft.com/documentation/services/virtual-network/) permite que você crie uma rede segura e persistente contendo os recursos necessários para sua solução. Uma rede virtual permite que você:

* Conecte recursos da nuvem juntos em uma rede privada (somente nuvem).

	![diagrama da configuração somente nuvem](media/hdinsight-extend-hadoop-virtual-network/cloud-only.png)

	Usar a Rede Virtual para vincular os serviços do Azure ao Azure HDInsight habilita os seguintes cenários:

	* **Invocar serviços ou trabalhos do HDInsight** de sites do Azure ou de serviços em execução em máquinas virtuais do Azure.

	* **Transferência direta de dados** entre o HDInsight e o Banco de Dados SQL do Azure, o SQL Server ou outra solução de armazenamento de dados em execução em uma máquina virtual.

	* **Combinar vários servidores HDInsight** em uma única solução. Um exemplo é o uso de um servidor HDInsight Storm para consumir dados de entrada para, em seguida, armazenar os dados processados em um servidor HDInsight HBase. Os dados brutos também podem ser armazenados em um servidor HDInsight Hadoop para análise futura usando MapReduce.

* Conecte seus recursos de nuvem à sua rede de datacenter local (site a site ou ponto a site), usando uma VPN (rede privada virtual).

	A configuração site a site permite conectar vários recursos do datacenter à rede virtual do Azure usando uma VPN de hardware ou o serviço de Roteamento e Acesso Remoto.

	![diagrama da configuração site a site](media/hdinsight-extend-hadoop-virtual-network/site-to-site.png)

	A configuração ponto a site permite conectar um recurso específico à rede virtual do Azure usando VPN de software.

	![diagrama da configuração ponto a site](media/hdinsight-extend-hadoop-virtual-network/point-to-site.png)

	Usar a Rede Virtual para vincular a nuvem e seu datacenter habilita cenários semelhantes à configuração somente nuvem. Porém, você não fica limitado a trabalhar com recursos na nuvem – também pode trabalhar com os recursos em seu datacenter.

	* **Transferência direta de dados** entre o HDInsight e seu datacenter. Um exemplo é usar o Sqoop para transferir dados para ou do SQL Server, ou a leitura de dados gerados por um aplicativo LOB (linha de negócios).

	* **Invocar serviços ou trabalhos do HDInsight** de um aplicativo LOB. Um exemplo é usar APIs do HBase Java para armazenar e recuperar dados de um cluster HDInsight HBase.

Para obter mais informações sobre os recursos, benefícios e capacidades das redes virtuais, consulte a [Visão geral da rede virtual do Azure](../virtual-network/virtual-networks-overview.md).

> [AZURE.NOTE] Você deve criar a Rede Virtual do Azure antes de provisionar um cluster HDInsight. Para obter mais informações, consulte [Tarefas de configuração de rede virtual](https://azure.microsoft.com/documentation/services/virtual-network/).

## Requisitos de Rede Virtual

> [AZURE.IMPORTANT] Criar um cluster HDInsight em uma Rede Virtual exige configurações de Rede Virtual específicas, que são descritas nesta seção.

###Redes virtuais baseadas em local

O Azure HDInsight dá suporte apenas a redes virtuais baseadas em local e atualmente não funciona com redes virtuais baseadas em grupo de afinidade.

###Sub-redes

É altamente recomendável que você crie uma única sub-rede para cada cluster do HDInsight.

###Rede virtual clássica ou v2

Os clusters baseados em Windows exigem uma Rede Virtual v1 (Clássica), enquanto que os clusters baseados em Linux exigem uma rede Virtual v2 (Gerenciador de Recursos do Azure). Se você não tiver o tipo correto de rede, ele não poderá ser usado durante a criação do cluster.

Se você tiver recursos em uma Rede Virtual que não pode ser usada pelo cluster que você planejar criar, é possível criar uma nova Rede Virtual que pode ser usada pelo cluster e conectá-la à Rede Virtual incompatível. Em seguida, você pode criar o cluster na versão de rede exigida, e ele poderá acessar os recursos na outra rede, pois as duas foram unidas. Para obter mais informações sobre como conectar Redes Virtuais clássicas e novas, veja [Conectando Redes Virtuais clássicas a Redes Virtuais novas](../virtual-network/virtual-networks-arm-asm-s2s.md).

###Redes virtuais protegidas

O serviço do HDInsight é um serviço gerenciado e requer acesso à Internet durante o provisionamento e a execução. Isso é necessário para que o Azure possa monitorar a integridade do cluster, iniciar o failover dos recursos de cluster, alterar o número de nós no cluster por meio de operações de escala, bem como outras tarefas de gerenciamento.

Se você precisar instalar o HDInsight em uma Rede Virtual protegida, será necessário permitir o acesso de entrada pela porta 443 para os seguintes endereços IP, que permitem ao Azure gerenciar o cluster HDInsight.

* 168\.61.49.99
* 23\.99.5.239
* 168\.61.48.131
* 138\.91.141.162

Permitir o acesso de entrada desses endereços por meio da porta 443 permitirá a instalação bem-sucedida do HDInsight em uma rede virtual protegida.

Os exemplos a seguir demonstram como criar um novo Grupo de Segurança de Rede que permite os endereços necessários e que aplica o grupo de segurança a uma sub-rede em sua Rede Virtual. Essas etapas pressupõem que você já tenha criado uma Rede Virtual e a sub-rede na qual deseja instalar o HDInsight.

__Usando o PowerShell do Azure__

    $vnetName = "Replace with your virtual network name"
    $resourceGroupName = "Replace with the resource group the virtual network is in"
    $subnetName = "Replace with the name of the subnet that HDInsight will be installed into"
    # Get the Virtual Network object
    $vnet = Get-AzureRmVirtualNetwork `
        -Name $vnetName `
        -ResourceGroupName $resourceGroupName
    # Get the region the Virtual network is in.
    $location = $vnet.Location
    # Get the subnet object
    $subnet = $vnet.Subnets | Where-Object Name -eq $subnetName
    # Create a new Network Security Group.
    # And add exemptions for the HDInsight health and management services.
    $nsg = New-AzureRmNetworkSecurityGroup `
        -Name "hdisecure" `
        -ResourceGroupName $resourceGroupName `
        -Location $location `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -name "hdirule1" `
            -Description "HDI health and management address 168.61.49.99" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.49.99" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 300 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule2" `
            -Description "HDI health and management 23.99.5.239" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "23.99.5.239" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 301 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule3" `
            -Description "HDI health and management 168.61.48.131" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "168.61.48.131" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 302 `
            -Direction Inbound `
        | Add-AzureRmNetworkSecurityRuleConfig `
            -Name "hdirule4" `
            -Description "HDI health and management 138.91.141.162" `
            -Protocol "*" `
            -SourcePortRange "*" `
            -DestinationPortRange "443" `
            -SourceAddressPrefix "138.91.141.162" `
            -DestinationAddressPrefix "VirtualNetwork" `
            -Access Allow `
            -Priority 303 `
            -Direction Inbound
    # Set the changes to the security group
    Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg
    # Apply the NSG to the subnet
    Set-AzureRmVirtualNetworkSubnetConfig `
        -VirtualNetwork $vnet `
        -Name $subnetName `
        -AddressPrefix $subnet.AddressPrefix `
        -NetworkSecurityGroupId $nsg

__Usando a CLI do Azure__

1. Use o seguinte comando para criar um novo grupo de segurança de rede chamado `hdisecure`. Substitua __RESOURCEGROUPNAME__ e __LOCATION__ pelo grupo de recursos que contém a Rede Virtual do Azure e pelo local (região) em que o grupo foi criado.

        azure network nsg create RESOURCEGROUPNAME hdisecure LOCATION
    
    Após a criação do grupo, você receberá informações sobre o novo grupo. Procure uma linha semelhante à seguinte e salve as informações de `/subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure`. Ele será usado em uma etapa posterior.
    
        data:    Id                              : /subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure

2. Use o seguinte para adicionar regras ao novo grupo de segurança de rede que permitem a comunicação de entrada na porta 443 por meio do serviço de integridade e gerenciamento do Azure HDInsight. Substitua __RESOURCEGROUPNAME__ pelo nome do grupo de recursos que contém a Rede Virtual do Azure.

        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule1 -p "*" -o "*" -u "443" -f "168.61.49.99" -e "VirtualNetwork" -c "Allow" -y 300 -r "Inbound"
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule2 -p "*" -o "*" -u "443" -f "23.99.5.239" -e "VirtualNetwork" -c "Allow" -y 301 -r "Inbound"
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule3 -p "*" -o "*" -u "443" -f "168.61.48.131" -e "VirtualNetwork" -c "Allow" -y 302 -r "Inbound"
        azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule4 -p "*" -o "*" -u "443" -f "138.91.141.162" -e "VirtualNetwork" -c "Allow" -y 303 -r "Inbound"

3. Após a criação das regras, use o seguinte para aplicar o novo grupo de segurança de rede a uma sub-rede. Substitua __RESOURCEGROUPNAME__ pelo nome do grupo de recursos que contém a Rede Virtual do Azure. Substitua __VNETNAME__ e __SUBNETNAME__ pelo nome da Rede Virtual do Azure e pela sub-rede que será usada durante a instalação do HDInsight.

        azure network vnet subnet set RESOURCEGROUPNAME VNETNAME SUBNETNAME -w "/subscriptions/GUID/resourceGroups/RESOURCEGROUPNAME/providers/Microsoft.Network/networkSecurityGroups/hdisecure"
    
    Após a conclusão desse comando, é possível instalar com êxito o HDInsight na Rede Virtual protegida, na sub-rede usada nessas etapas.

> [AZURE.IMPORTANT] O uso das etapas acima abre o acesso somente ao serviço de integridade e gerenciamento do HDInsight na nuvem do Azure. Isso permite uma instalação bem-sucedida de um cluster HDInsight na sub-rede; no entanto, o acesso ao cluster HDInsight de fora da Rede Virtual é bloqueado por padrão. Você terá de adicionar mais regras do Grupo de Segurança de Rede se desejar habilitar o acesso de fora da Rede Virtual.
>
> Por exemplo, para permitir o acesso SSH da Internet, você precisará adicionar uma regra semelhante à seguinte:
>
> * Azure PowerShell - ```Add-AzureRmNetworkSecurityRuleConfig -Name "SSSH" -Description "SSH" -Protocol "*" -SourcePortRange "*" -DestinationPortRange "22" -SourceAddressPrefix "*" -DestinationAddressPrefix "VirtualNetwork" -Access Allow -Priority 304 -Direction Inbound```
> * CLI do Azure - ```azure network nsg rule create RESOURCEGROUPNAME hdisecure hdirule4 -p "*" -o "*" -u "22" -f "*" -e "VirtualNetwork" -c "Allow" -y 304 -r "Inbound"```

Para obter mais informações sobre os Grupos de Segurança de Rede, confira [Visão geral dos Grupos de Segurança de Rede](../virtual-network/virtual-networks-nsg.md). Para obter mais informações sobre como controlar o roteamento em uma Rede Virtual do Azure, veja [Rotas e encaminhamento IP definidos pelo usuário](../virtual-network/virtual-networks-udr-overview.md).

##<a id="tasks"></a>Tarefas e informações

Esta seção contém informações sobre tarefas comuns e informações que podem ser necessárias ao usar o HDInsight com uma rede virtual.

###Determinar o FQDN

Um FQDN (nome de domínio totalmente qualificado) será atribuído ao cluster HDInsight para a interface da Rede Virtual. Esse é o endereço que você deve usar ao se conectar ao cluster por meio de outros recursos na rede virtual. Para determinar o FQDN, use a seguinte URL para consultar o serviço de gerenciamento Ambari:

	https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/services/<servicename>/components/<componentname>

> [AZURE.NOTE] Para obter mais informações sobre como usar o Ambari com o HDInsight, consulte [Monitorar clusters Hadoop no HDInsight usando a API do Ambari](hdinsight-monitor-use-ambari-api.md).

Você deve especificar o nome do cluster e um serviço e componente em execução no cluster, como o gerenciador de recursos YARN.

> [AZURE.NOTE] Os dados retornados são um documento JSON (JavaScript Object Notation) que contém muitas informações sobre o componente. Para extrair apenas o FQDN, você deve usar um analisador JSON para recuperar o valor `host_components[0].HostRoles.host_name`.

Por exemplo, para retornar o FQDN de um cluster Hadoop no HDInsight, você pode usar um dos métodos a seguir para recuperar os dados para o gerenciador de recursos YARN:

* [PowerShell do Azure](../powershell-install-configure.md)

		$ClusterDnsName = <clustername>
		$Username = <cluster admin username>
		$Password = <cluster admin password>
		$DnsSuffix = ".azurehdinsight.net"
		$ClusterFQDN = $ClusterDnsName + $DnsSuffix

		$webclient = new-object System.Net.WebClient
		$webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)

		$Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/services/yarn/		components/resourcemanager"
		$Response = $webclient.DownloadString($Url)
		$JsonObject = $Response | ConvertFrom-Json
		$FQDN = $JsonObject.host_components[0].HostRoles.host_name
		Write-host $FQDN

* [cURL](http://curl.haxx.se/) e [jq](http://stedolan.github.io/jq/)

		curl -G -u <username>:<password> https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/services/yarn/components/resourcemanager | jq .host_components[0].HostRoles.host_name

###Conectando-se ao HBase

Para se conectar ao HBase remotamente usando a API do Java, você deve determinar os endereços de quorum ZooKeeper para o cluster HBase e especificá-los no seu aplicativo.

Para obter o endereço de quorum ZooKeeper, use um dos seguintes métodos para consultar o serviço de gerenciamento Ambari:

* [PowerShell do Azure](../powershell-install-configure.md)

		$ClusterDnsName = <clustername>
		$Username = <cluster admin username>
		$Password = <cluster admin password>
		$DnsSuffix = ".azurehdinsight.net"
		$ClusterFQDN = $ClusterDnsName + $DnsSuffix

		$webclient = new-object System.Net.WebClient
		$webclient.Credentials = new-object System.Net.NetworkCredential($Username, $Password)

		$Url = "https://" + $ClusterFQDN + "/ambari/api/v1/clusters/" + $ClusterFQDN + "/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum"
        $Response = $webclient.DownloadString($Url)
        $JsonObject = $Response | ConvertFrom-Json
        Write-host $JsonObject.items[0].properties.'hbase.zookeeper.quorum'

* [cURL](http://curl.haxx.se/) e [jq](http://stedolan.github.io/jq/)

		curl -G -u <username>:<password> "https://<clustername>.azurehdinsight.net/ambari/api/v1/clusters/<clustername>.azurehdinsight.net/configurations?type=hbase-site&tag=default&fields=items/properties/hbase.zookeeper.quorum" | jq .items[0].properties[]

> [AZURE.NOTE] Para obter mais informações sobre como usar o Ambari com o HDInsight, consulte [Monitorar clusters Hadoop no HDInsight usando a API do Ambari](hdinsight-monitor-use-ambari-api.md).

Uma vez que as informações estejam reunidas, use-as em seu aplicativo cliente.

Por exemplo, para um aplicativo Java que usa a API do HBase, você adicionaria um arquivo **hbase-site.htm** ao projeto e especificaria as informações de quorum no arquivo da seguinte maneira:

```
<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>zookeeper0.address,zookeeper1.address,zookeeper2.address</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
</configuration>
```

###Verificar a conectividade de rede

Alguns serviços, como o SQL Server, podem limitar conexões de rede. Isso impedirá que HDInsight trabalhe com êxito com esses serviços.

Se você encontrar problemas ao acessar um serviço do HDInsight, consulte a documentação para o serviço para garantir que você habilitou o acesso à rede. Você também pode verificar o acesso à rede criando uma máquina virtual do Azure na mesma rede virtual e usar utilitários de cliente para verificar se a máquina virtual consegue se conectar ao serviço através da rede virtual.

##<a id="nextsteps"></a>Próximas etapas

Os exemplos a seguir demonstram como usar o HDInsight com a Rede Virtual do Azure:

* [Analisar dados de sensor com o Storm e o HBase no HDInsight](hdinsight-storm-sensor-data-analysis.md): demonstra como configurar um cluster Storm e HBase em uma rede virtual e como gravar dados remotamente no HBase pelo Storm.

* [Provisionar clusters Hadoop no HDInsight](hdinsight-hadoop-provision-linux-clusters.md): fornece informações sobre como provisionar clusters Hadoop, inclusive informações sobre como usar a Rede Virtual do Azure.

* [Usar o Sqoop com o Hadoop no HDInsight](hdinsight-use-sqoop-mac-linux.md): fornece informações sobre como usar o Sqoop para transferir dados com o SQL Server em uma rede virtual.

Para saber mais sobre redes virtuais do Azure, consulte [Visão geral da Rede Virtual do Azure](../virtual-network/virtual-networks-overview.md).

<!---HONumber=AcomDC_0504_2016-->