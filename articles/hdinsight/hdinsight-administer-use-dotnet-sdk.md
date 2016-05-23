<properties
	pageTitle="Gerenciar clusters Hadoop no HDInsight com o SDK do .NET | Microsoft Azure"
	description="Saiba como realizar tarefas administrativas para os clusters Hadoop no HDInsight usando o SDK do .NET do HDInsight."
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	tags="azure-portal"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/02/2016"
	ms.author="jgao"/>

# Gerenciar clusters Hadoop no HDInsight usando o SDK do .NET

[AZURE.INCLUDE [seletor](../../includes/hdinsight-portal-management-selector.md)]

Saiba como gerenciar clusters HDInsight usando o [SDK do .NET do HDInsight](https://msdn.microsoft.com/library/mt271028.aspx).


**Pré-requisitos**

Antes de começar este artigo, você deve ter o seguinte:

- **Uma assinatura do Azure**. Consulte [Obter avaliação gratuita do Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).


##Conectar-se ao Azure HDInsight

Você precisará dos seguintes pacotes Nuget:

	Install-Package Microsoft.Azure.Common.Authentication -Pre
	Install-Package Microsoft.Azure.Management.HDInsight -Pre

O exemplo de código a seguir mostra como se conectar ao Azure antes que você possa administrar os clusters HDInsight em sua assinatura do Azure.

	using System;
	using System.Security;
	using Microsoft.Azure;
	using Microsoft.Azure.Common.Authentication;
	using Microsoft.Azure.Common.Authentication.Factories;
	using Microsoft.Azure.Common.Authentication.Models;
	using Microsoft.Azure.Management.HDInsight;
	using Microsoft.Azure.Management.HDInsight.Models;

	namespace HDInsightManagement
	{
		class Program
		{
			private static HDInsightManagementClient _hdiManagementClient;
			private static Guid SubscriptionId = new Guid("<Your Azure Subscription ID>");

			static void Main(string[] args)
			{
				var tokenCreds = GetTokenCloudCredentials();
				var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);

				_hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);

				// insert code here

				System.Console.WriteLine("Press ENTER to continue");
				System.Console.ReadLine();
			}

			public static TokenCloudCredentials GetTokenCloudCredentials(string username = null, SecureString password = null)
			{
				var authFactory = new AuthenticationFactory();

				var account = new AzureAccount { Type = AzureAccount.AccountType.User };

				if (username != null && password != null)
					account.Id = username;

				var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];

				var accessToken =
					authFactory.Authenticate(account, env, AuthenticationFactory.CommonAdTenant, password, ShowDialog.Auto)
						.AccessToken;

				return new TokenCloudCredentials(accessToken);
			}

			public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(TokenCloudCredentials creds, Guid subId)
			{
				return new TokenCloudCredentials(subId.ToString(), creds.Token);
			}
		}
	}

Você deverá ver um aviso ao executar este programa. Se não desejar ver o aviso, confira [Criar aplicativos .NET do HDInsight com autenticação não interativa](hdinsight-create-non-interactive-authentication-dotnet-applications.md).

##Criar clusters

Veja [Criar clusters baseados em Linux no HDInsight usando o SDK do .NET](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md)

##Listar clusters

O trecho de código a seguir lista os clusters e algumas propriedades:

    var results = _hdiManagementClient.Clusters.List();
    foreach (var name in results.Clusters) {
        Console.WriteLine("Cluster Name: " + name.Name);
        Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
        Console.WriteLine("\t Cluster location: " + name.Location);
        Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
    }

##Excluir clusters

Use o seguinte trecho de código para excluir um cluster de forma síncrona ou assíncrona:

    _hdiManagementClient.Clusters.Delete("<Resource Group Name>", "<Cluster Name>");
    _hdiManagementClient.Clusters.DeleteAsync("<Resource Group Name>", "<Cluster Name>");
            
##Dimensionar clusters
O recurso de dimensionamento de clusters permite que você altere o número de nós de trabalhador usados por um cluster em execução no Azure HDInsight sem precisar recriar o cluster.

>[AZURE.NOTE] Somente clusters HDInsight versão 3.1.3 ou superior são compatíveis. Se não tiver certeza quanto à versão de seu cluster, você poderá verificar a página Propriedades. Consulte [Listar e mostrar clusters](hdinsight-administer-use-portal-linux.md#list-and-show-clusters).

O impacto da alteração do número de nós de dados em cada tipo de cluster com suporte do HDInsight:

- O Hadoop

	Você pode aumentar continuamente o número de nós de trabalhador em um cluster Hadoop em execução sem afetar os trabalhos pendentes ou em execução. Novos trabalhos também podem ser enviados enquanto a operação está em andamento. Falhas em uma operação de dimensionamento são normalmente manipuladas para que o cluster sempre seja deixado em um estado funcional.

	Quando um cluster Hadoop é reduzido verticalmente pela diminuição do número de nós de dados, alguns dos serviços no cluster são reiniciados. Isso faz com que todos os trabalhos em execução e pendentes falhem após a conclusão da operação de dimensionamento. Você pode, no entanto, reenviar os trabalhos quando a operação for concluída.

- HBase

	Você pode adicionar ou remover diretamente nós do cluster HBase enquanto ele é executado. Servidores Regionais são equilibrados automaticamente em alguns minutos após o término da operação de dimensionamento. No entanto, você pode equilibrar manualmente os servidores regionais fazendo logon no nó de cabeçalho do cluster e executando os seguintes comandos em uma janela de prompt de comando:

		>pushd %HBASE_HOME%\bin
		>hbase shell
		>balancer

- Storm

	Você pode adicionar ou remover nós de dados continuamente para seu cluster Strom enquanto ele é executado. Mas, após a conclusão bem-sucedida da operação de dimensionamento, você precisará redistribuir a topologia.

	A redistribuição pode ser feita de duas maneiras:

	* Interface da Web Storm
	* Ferramenta CLI (interface de linha de comando)

	Consulte a [documentação do Apache Storm](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html) para obter mais detalhes.

	A IU da Web do Storm está disponível no cluster HDInsight:

	![hdinsight storm dimensionar novo balanceamento](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.storm.rebalance.png)

	Aqui está um exemplo de como usar o comando CLI para reequilibrar a topologia do Storm:

		## Reconfigure the topology "mytopology" to use 5 worker processes,
		## the spout "blue-spout" to use 3 executors, and
		## the bolt "yellow-bolt" to use 10 executors

		$ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

O trecho de código a seguir mostra como redimensionar um cluster de forma síncrona ou assíncrona:

    _hdiManagementClient.Clusters.Resize("<Resource Group Name>", "<Cluster Name>", <New Size>);   
    _hdiManagementClient.Clusters.ResizeAsync("<Resource Group Name>", "<Cluster Name>", <New Size>);   
	

##Conceder/revogar acesso

Os clusters HDInsight têm os seguintes serviços Web HTTP (todos esses serviços têm pontos de extremidade RESTful):

- ODBC
- JDBC
- Ambari
- Oozie
- Templeton


Por padrão, esses serviços são concedidos para acesso. Você pode revogar/conceder o acesso. Para revogar:

	var httpParams = new HttpSettingsParameters
	{
		HttpUserEnabled = false,
		HttpUsername = "admin",
		HttpPassword = "*******",
	};
	_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);

Para conceder:

	var httpParams = new HttpSettingsParameters
	{
		HttpUserEnabled = enable,
		HttpUsername = "admin",
		HttpPassword = "*******",
	};
	_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);


>[AZURE.NOTE] Ao conceder/revogar o acesso, você redefinirá o nome de usuário de cluster e a senha.

Isso também pode ser feito por meio do Portal. Consulte [Administrar o HDInsight usando o portal do Azure][hdinsight-admin-portal].

##Atualizar credenciais de usuário HTTP

É o mesmo procedimento para acessar [HTTP Conveder/Revogar](#grant/revoke-access). Se o cluster recebeu o acesso HTTP, você deverá revogá-lo. E, em seguida, conceder acesso com novas credenciais de usuário HTTP.


##Encontrar a conta de armazenamento padrão

O trecho de código a seguir demonstra como obter o nome da conta de armazenamento padrão e a chave da conta de armazenamento padrão de um cluster.

	var results = _hdiManagementClient.Clusters.GetClusterConfigurations(<Resource Group Name>, <Cluster Name>, "core-site");
	foreach (var key in results.Configuration.Keys)
	{
	    Console.WriteLine(String.Format("{0} => {1}", key, results.Configuration[key]));
	}


##Enviar trabalhos

**Enviar trabalhos MapReduce**

Veja [Executar amostras de MapReduce do Hadoop no HDInsight](hdinsight-hadoop-run-samples-linux.md).

**Enviar trabalhos Hive**

Veja [Executar consultas do Hive usando o SDK do .NET](hdinsight-hadoop-use-hive-dotnet-sdk.md).

**Enviar trabalhos Pig**

Veja [Executar trabalhos do Pig usando o SDK do .NET](hdinsight-hadoop-use-pig-dotnet-sdk.md).

**Para enviar trabalhos de Sqoop**

Consulte [Usar o Sqoop com o HDInsight](hdinsight-hadoop-use-sqoop-dotnet-sdk.md).

**Para enviar trabalhos de Oozie**

Consulte [Usar o Oozie com Hadoop para definir e executar um fluxo de trabalho no HDInsight](hdinsight-use-oozie-linux-mac.md).

##Carregar dados no armazenamento de Blob do Azure
Consulte [Carregar dados no HDInsight][hdinsight-upload-data].


## Consulte também
* [Documentação de referência do SDK do .NET do HDInsight](https://msdn.microsoft.com/library/mt271028.aspx)
* [Administrar o HDInsight usando o Portal do Azure][hdinsight-admin-portal]
* [Administrar o HDInsight usando uma interface de linha de comando][hdinsight-admin-cli]
* [Criar clusters HDInsight][hdinsight-provision]
* [Carregar dados no HDInsight][hdinsight-upload-data]
* [Introdução ao Azure HDInsight][hdinsight-get-started]


[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-get-started]: hdinsight-hadoop-linux-tutorial-get-started.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-provision-custom-options]: hdinsight-provision-clusters.md#configuration
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-portal-linux.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md

<!---HONumber=AcomDC_0504_2016-->