<properties 
   pageTitle="Projeto de banco de dados SQL para continuidade de negócios" 
   description="Diretrizes para escolher nesta seção, orientações serão fornecida para como escolher quais recursos de BCDR devem ser usados e quando. Isso inclui descrições de o que você obtém automaticamente usando o banco de dados SQL."
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-management" 
   ms.date="04/25/2016"
   ms.author="elfish"/>

#Projeto para continuidade dos negócios

Para criar seu aplicativo para continuidade dos negócios, é necessário que você responda às seguintes perguntas:

1. Qual recurso de continuidade dos negócios é adequado para proteger meu aplicativo contra interrupções?
2. Qual o nível de redundância e a topologia de replicação eu devo usar?

##Quando usar a restauração geográfica

O banco de dados SQL fornece uma proteção básica interna de cada banco de dados por padrão. Isso é feito ao armazenar os backups de banco de dados no armazenamento do Azure com redundância geográfica (GRS). Se você escolher esse método, nenhuma configuração especial ou alocação adicional de recursos é necessária. Com esses backups, é possível recuperar o banco de dados em qualquer região usando o comando de restauração geográfica. Use a seção [Recuperação de uma interrupção](sql-database-disaster-recovery.md) para obter os detalhes do uso de restauração geográfica para recuperar seu aplicativo.

Você deve usar a proteção interna caso seu aplicativo atenda aos seguintes critérios:

1. Ele não é considerado crítico. Ele não tem um SLA vinculado, portanto, o tempo de inatividade de 24 horas ou mais não resultará em responsabilidade financeira.
2. A taxa de alteração de dados é baixa (por exemplo, transações por hora). O RPO de 1 hora não resultará em uma grande perda de dados.
3. O aplicativo é de baixo custo e não é possível justificar o custo adicional de replicação geográfica 

> [AZURE.NOTE] A restauração geográfica não aloca previamente a capacidade de computação em nenhuma região específica para restaurar bancos de dados ativos do backup durante a interrupção. O serviço irá gerenciar a carga de trabalho associada às solicitações de restauração geográfica de maneira a minimizar o impacto sobre os bancos de dados existentes nessa região e suas demandas de capacidade terão prioridade. Portanto, o tempo de recuperação do banco de dados dependerá de quantos outros bancos de dados serão recuperados na mesma região ao mesmo tempo.

##Quando usar a replicação geográfica

A replicação geográfica cria um banco de dados de réplica (secundário) em uma região diferente do principal. Ele garante que seu banco de dados terá os dados e recursos de computação necessários para oferecer suporte às cargas de trabalho do aplicativo após a recuperação. Consulte a seção [Recuperação de uma interrupção](sql-database-disaster-recovery.md) para usar o failover para recuperar seu aplicativo.

Você deve usar a replicação geográfica caso seu aplicativo atenda aos seguintes critérios:

1. Ele é de crítico. Ele tem um SLA vinculado com RTO e RPO agressivos. A perda de dados e a disponibilidade resultarão em responsabilidade financeira. 
2. A taxa de alteração de dados é alta (por exemplo, transações por minuto ou segundos). O RPO de 1 h associado à proteção padrão provavelmente resultará em perda de dados inaceitável.
3. O custo associado ao uso da replicação geográfica é significativamente menor do que a responsabilidade financeira potencial e as perdas associadas do negócio.


> [AZURE.NOTE] A replicação geográfica também oferece suporte a acesso somente leitura ao banco de dados secundário, proporcionando capacidade adicional para as cargas de trabalho somente leitura.

##Como habilitar a replicação geográfica

Você pode habilitar replicação geográfica usando o Portal clássico do Azure ou chamando a API REST ou comando do PowerShell.

###Portal do Azure

[AZURE.VIDEO sql-database-enable-geo-replication-in-azure-portal]

1. Faça logon no [portal do Azure](https://portal.Azure.com)
2. No lado esquerdo da tela, selecione **PROCURAR** e selecione **Bancos de Dados SQL**
3. Navegue até a folha de banco de dados, selecione o **Mapa de replicação geográfica** e clique em **Configurar a replicação geográfica**.
4. Navegue até a folha de replicação geográfica. Selecione a região de destino. 
5. Navegue até a folha Criar Secundário. Selecione um servidor existente na região de destino ou crie um novo.
6. Selecione o tipo de secundário (*Legível* ou *Não legível*)
7. Clique em **Criar** para concluir a configuração

> [AZURE.NOTE] A região pareada de DR na folha de replicação geográfica será marcada como *recomendada*, mas você pode escolher uma região diferente.


###PowerShell

Use o cmdlet [New-AzureRmSqlDatabaseSecondary](https://msdn.microsoft.com/library/mt603689.aspx) do PowerShell para automatizar a configuração de replicação geográfica. Este comando é síncrono um retorna quando os bancos de dados primários e secundários estiverem sincronizados.

Para configurar a replicação geográfica com um secundário não legível:
		
    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" -AllowConnections "None"

Para criar a replicação geográfica com um secundário legível:

    $database = Get-AzureRmSqlDatabase –DatabaseName "mydb"
    $secondaryLink = $database | New-AzureRmSqlDatabaseSecondary –PartnerResourceGroupName "rg2" –PartnerServerName "srv2" -AllowConnections "All"
		 

###API REST 

Use [Criar API](https://msdn.microsoft.com/library/mt163685.aspx) de banco de dados com *createMode* definido como *NonReadableSecondary* ou *secundário* para criar programaticamente um banco de dados secundário de replicação geográfica.

Esse API é assíncrono. Depois de retornar, use o API [Obter replicação do link](https://msdn.microsoft.com/library/mt600778.aspx) para verificar o status dessa operação. O campo *replicationState* do corpo da resposta terá o valor CATCH\_UP quando a operação for concluída.


##Como escolher a configuração de failover 

Ao projetar seu aplicativo para continuidade dos negócios, você deve considerar várias opções de configuração. A escolha dependerá da topologia de implantação do aplicativo e quais partes de seus aplicativos são mais vulneráveis a uma interrupção. Para diretrizes, consulte [Criando soluções de nuvem para recuperação de desastres usando replicação geográfica](sql-database-designing-cloud-solutions-for-disaster-recovery.md).

<!---HONumber=AcomDC_0504_2016-->