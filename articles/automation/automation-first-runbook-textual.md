<properties
    pageTitle="O meu primeiro runbook de Fluxo de Trabalho do PowerShell na Automação do Azure | Microsoft Azure"
    description="Tutorial que orienta você pela criação, teste e publicação de um runbook textual simples usando o Fluxo de Trabalho do PowerShell."
    services="automation"
    documentationCenter=""
    authors="mgoedtel"
    manager="jwhit"
    editor=""
	keywords="fluxo de trabalho do powershell, exemplos de fluxo de trabalho do powershell, fluxos de trabalho do powershell" />
<tags
    ms.service="automation"
    ms.workload="tbd"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="05/10/2016"
    ms.author="magoedte;bwren"/>

# Meu primeiro runbook de Fluxo de Trabalho do PowerShell

> [AZURE.SELECTOR] - [Graphical](automation-first-runbook-graphical.md) - [PowerShell Workflow](automation-first-runbook-textual.md) - [PowerShell](automation-first-runbook-textual-PowerShell.md)

Este tutorial orienta você durante a criação de um [runbook de Fluxo de Trabalho do PowerShell](automation-runbook-types.md#powerShell-workflow-runbooks) na Automação do Azure. Começaremos com um runbook simples, que testaremos e publicaremos enquanto explicamos como acompanhar o status do trabalho do runbook. Em seguida, modificaremos o runbook para gerenciar recursos do Azure, nesse caso, iniciando uma máquina virtual do Azure. Depois, tornaremos o runbook mais robusto adicionando parâmetros de runbook.

## Pré-requisitos

Para concluir este tutorial, você precisará do seguinte.

-	do Microsoft Azure. Se você ainda não tiver uma, poderá [ativar os benefícios de assinante do MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou <a href="/pricing/free-account/" target="_blank">[inscrever-se em uma conta gratuita](https://azure.microsoft.com/free/).
-	[Conta de automação](automation-security-overview.md) para manter o runbook e se autenticar nos recursos do Azure. Esta conta deve ter permissão para iniciar e parar a máquina virtual.
-	Uma máquina virtual do Azure. Vamos parar e iniciar esse computador, portanto ele não deve ser de produção.

## Etapa 1: criar o novo runbook

Começaremos criando um runbook simples que exibe o texto *Olá mundo*.

1.	No Portal do Azure, abra sua conta de Automação. A página da conta de Automação fornece uma exibição rápida dos recursos nessa conta. Você já deve ter alguns ativos. A maioria deles são os módulos que são incluídos automaticamente em uma nova conta de Automação. Você também deve ter o ativo de credencial que é mencionado nos [pré-requisitos](#prerequisites).
2.	Clique no bloco **Runbooks** para abrir a lista de runbooks.<br> ![Controle de runbooks](media/automation-first-runbook-textual/runbooks-control.png)
3.	Crie um novo runbook clicando no botão **Adicionar um runbook** e em **Criar um novo runbook**.
4.	Atribua o nome *MyFirstRunbook-Workflow* ao runbook.
5.	Nesse caso, vamos criar um [runbook de Fluxo de Trabalho do PowerShell](automation-runbook-types.md#powerShell-workflow-runbooks), portanto, escolha **Fluxo de Trabalho do PowerShell** como o **tipo de Runbook**.<br> ![Novo runbook](media/automation-first-runbook-textual/new-runbook.png)
6.	Clique em **criar** para criar o runbook e abrir o editor textual.

## Etapa 2 - adicionar código ao runbook

Você pode digitar o código diretamente no runbook, ou você pode selecionar cmdlets, runbooks, e ativos do controle de biblioteca e adicioná-los ao runbook com quaisquer parâmetros relacionados. Para este passo a passo, digitaremos diretamente no runbook.

1.	Nosso runbook está atualmente vazio, com apenas o a palavra-chave necessária do *fluxo de trabalho*, o nome do nosso runbook, e as chaves que encerrarão o fluxo de trabalho inteiro.<br>![Controle de runbooks](media/automation-first-runbook-textual/empty-runbook.png)
2.	Digitar *Write-Output "Hello World".* entre as chaves.<br>![Olá mundo](media/automation-first-runbook-textual/hello-world.png)
3.	Salve o runbook clicando em **Salvar**.<br> ![Salvar runbook](media/automation-first-runbook-textual/runbook-edit-toolbar-save.png)

## Etapa 3: testar o runbook

Antes que publicamos o runbook para disponibilizá-lo na produção, queremos testá-lo para garantir que ele funciona corretamente. Quando você testa um runbook, executa sua versão de **Rascunho** e vê sua saída interativamente.

1.	Clique em **Painel de teste** para abrir o Painel de teste.<br> ![Painel de teste](media/automation-first-runbook-textual/runbook-edit-toolbar-test-pane.png)
2.	Clique em **Iniciar** para iniciar o teste. Essa deve ser a única opção habilitada.
3.	Um [trabalho de runbook](automation-runbook-execution.md) é criado e seu status é exibido. O status do trabalho iniciará como *Na fila*, indicando que ele está aguardando um runbook worker na nuvem ficar disponível. Ele mudará para *Iniciando* quando um runbook worker reivindicar o trabalho e para *Executando* quando o runbook realmente começar a ser executado.  
4.	Quando o trabalho do runbook é concluído, sua saída é exibida. Em nosso caso, deveremos ver *Olá mundo*.<br> ![Olá mundo](media/automation-first-runbook-textual/test-output-hello-world.png)
5.	Feche o Painel de teste para retornar à tela.

## Etapa 4: publicar e iniciar o runbook

O runbook que acabamos de criar ainda está em Modo de rascunho. Precisamos publicá-lo antes que possamos executá-lo na produção. Quando você publica um runbook, substitui a versão Publicada existente pela versão de Rascunho. Em nosso caso, não temos uma versão Publicada ainda porque que acabamos de criar o runbook.

1.	Clique em **Publicar** para publicar o runbook e em **Sim** quando solicitado.<br> ![Publicar](media/automation-first-runbook-textual/runbook-edit-toolbar-publish.png)
2.	Se você rolar para a esquerda para exibir o runbook no painel **Runbooks** agora, ele mostrará um **Status de Criação** de **Publicado**.
3.	Role para a direita para exibir o painel de **MyFirstRunbook-Workflow**. As opções na parte superior nos permitem iniciar o runbook, agendá-lo para iniciar em algum momento no futuro ou criar um [webhook](automation-webhooks.md) para que ele possa ser iniciado por meio de uma chamada de HTTP.
4.	Queremos apenas iniciar o runbook, então clique em **Iniciar** e em **Sim** quando solicitado.<br> ![Iniciar runbook](media/automation-first-runbook-textual/runbook-toolbar-start.png)
5.	Um painel de trabalho é aberto para o trabalho de runbook que acabamos de criar. Podemos fechar esse painel, mas nesse caso, o deixaremos aberto para que possamos acompanhar o progresso do trabalho.
6.	O status do trabalho é mostrado em **Resumo do trabalho** e corresponde aos status que vimos quando testamos o runbook.<br> ![Resumo do trabalho](media/automation-first-runbook-textual/job-pane-summary.png)
7.	Assim que o status do runbook mostrar *Concluído*, clique em **Saída**. O painel Saída é aberto e podemos ver nosso *Olá mundo*.<br> ![Resumo do trabalho](media/automation-first-runbook-textual/job-pane-output.png)  
8.	Feche o painel Saída.
9.	Clique em **Fluxos** para abrir o painel Fluxos do trabalho do runbook. Devemos ver apenas *Olá mundo* no fluxo de saída, mas isso pode mostrar outros fluxos de um trabalho do runbook como Detalhado e Erro se o runbook gravar neles.<br>![Resumo do trabalho](media/automation-first-runbook-textual/job-pane-streams.png)
10.	Feche o painel Fluxos e o painel Trabalho para retornar ao painel MyFirstRunbook.
11.	Clique em **Trabalhos** para abrir o painel de trabalhos para este runbook. Ele lista todos os trabalhos criados por esse runbook. Devemos ver apenas um trabalho listado, já que executamos o trabalho apenas uma vez.<br> ![Trabalhos](media/automation-first-runbook-textual/runbook-control-jobs.png)
12.	Você pode clicar neste trabalho para abrir o mesmo painel Trabalho que exibimos quando iniciamos o runbook. Isso permite que você volte no tempo e veja os detalhes de qualquer trabalho que foi criado para um determinado runbook.

## Etapa 5: adicionar autenticação para gerenciar recursos do Azure

Testamos e publicamos nosso runbook, mas até o momento ele não faz nada útil. Gostaríamos que ele gerenciasse recursos do Azure. No entanto, ele não será capaz de fazer isso a menos que o autentiquemos usando as credenciais mencionadas nos [pré-requisitos](#prerequisites). Fazemos isso com o cmdlet **Add-AzureAccount**.

1.	Abra o editor gráfico clicando em **Editar** no painel MyFirstRunbook-Workflow.<br> ![Editar runbook](media/automation-first-runbook-textual/runbook-toolbar-edit.png)
2.	Já não precisamos da linha **Write-Output**, então vamos continuar e excluí-la.
3.	Posicione o cursor em uma linha em branco entre as chaves.
4.	No controle de biblioteca, expanda **Ativos** e **Credenciais**.
5.	Clique com o botão direito do mouse na sua credencial e clique em **Adicionar à tela**. Isso adiciona uma atividade **Get-AutomationPSCredential** para sua credencial.
6.	Na frente do **Get-AutomationPSCredential**, digite *$Credential =* para atribuir a credencial a uma variável.
7.	Na próxima linha, digite *Add-AzureAccount -Credential $Credential*. <br> ![Autenticar](media/automation-first-runbook-textual/authentication.png)
8.	Clique no **Painel de teste** para que possamos testar o runbook.
9.	Clique em **Iniciar** para iniciar o teste. Quando for concluído, você deve receber a saída semelhante à seguinte que retorna as informações para o usuário na credencial. Isso confirma que a credencial é válida.<br> ![Autenticar](media/automation-first-runbook-textual/authentication-test.png)

## Etapa 6: adicionar código para iniciar uma máquina virtual

Agora que nosso runbook está se autenticando em nossa assinatura do Azure, podemos gerenciar recursos. Adicionaremos um comando para iniciar uma máquina virtual. Você pode escolher qualquer máquina virtual na sua assinatura do Azure e, por enquanto, embutiremos esse nome no cmdlet.

1.	Após *Add-AzureAccount*, digite *Start-AzureVM-nome 'VMName' - ServiceName 'VMServiceName'* fornecendo o nome e o nome do serviço da máquina virtual a ser iniciada.<br>![Autenticar](media/automation-first-runbook-textual/start-azurevm.png)
2.	Salve o runbook e, para que possamos testá-lo, clique em **Painel de teste**.
3.	Clique em **Iniciar** para iniciar o teste. Quando for concluído, verifique se a máquina virtual foi iniciada.

## Etapa 7: adicionar um parâmetro de entrada ao runbook

No momento, nosso runbook inicia a máquina virtual que codificamos no runbook, mas ele seria mais útil pudéssemos especificar a máquina virtual quando o runbook é iniciado. Agora adicionaremos parâmetros de entrada ao runbook para fornecer essa funcionalidade.

1.	Adicione parâmetros para *VMName* e *VMServiceName* ao runbook e use essas variáveis com o cmdlet **Start-AzureVM** como na imagem a seguir. <br> ![Autenticar](media/automation-first-runbook-textual/params.png)
2.	Salve o runbook e abra o Painel de teste. Observe que agora você pode fornecer valores para as duas variáveis de entrada que serão usadas no teste.
3.	Feche o Painel de teste.
4.	Clique em **Publicar** para publicar a nova versão do runbook.
5.	Pare a máquina virtual que você iniciou na etapa anterior.
6.	Clique em **Iniciar** para iniciar o runbook. Digite o **VMName** e o **VMServiceName** da máquina virtual que você iniciará.<br> ![Iniciar Runbook](media/automation-first-runbook-textual/start-runbook-input-params.png)

7.	Quando o runbook for concluído, verifique se a máquina virtual foi iniciada.

## Artigos relacionados

-	[O meu primeiro runbook gráfico](automation-first-runbook-graphical.md)
-	[Meu primeiro runbook do PowerShell](automation-first-runbook-textual-PowerShell.md)

<!---HONumber=AcomDC_0511_2016-->