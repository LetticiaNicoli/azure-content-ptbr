<properties 
	pageTitle="Application Insights: detecção proativa" 
	description="O Application Insights executa uma análise profunda de telemetria do seu aplicativo e o avisará sobre possíveis problemas de desempenho." 
	services="application-insights" 
    documentationCenter="windows"
	authors="antonfrMSFT" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="01/15/2016" 
	ms.author="awills"/>

#  Application Insights: detecção proativa

*O Application Insights está em modo de visualização.*


O Application Insights executa uma análise profunda de telemetria do seu aplicativo e o avisará sobre possíveis problemas de desempenho. Provavelmente você está lendo este artigo porque recebeu um dos nossas alertas proativos por email.


## O que é a Detecção Proativa?

A Detecção Proativa detecta anomalias de desempenho em seu aplicativo analisando a telemetria que ele envia ao Application Insights.

Em particular, ele encontra problemas de desempenho que afetam apenas alguns dos seus usuários, ou só afetam seus usuários em alguns casos.

Por exemplo, ele pode notificá-lo se suas páginas do aplicativo carregarem muito mais lentamente em um tipo de navegador do que em outros, ou se solicitações forem atendidas de modo mais lento de um servidor específico. Ela também pode descobrir problemas associados com combinações de propriedades, como carregamentos de página lentos em uma área geográfica em determinados horários do dia.

Anomalias como essas são muito difíceis de detectar apenas inspecionando os dados, mas são mais comuns do que você imagina. Elas geralmente só surgem quando seus clientes reclamam. Nesse momento, é tarde demais: os usuários afetados já estão mudando para a concorrência!

Atualmente, nossos algoritmos Examinam os tempos de carregamento de página, tempos de resposta de solicitação no servidor e os tempos de resposta de dependência.

Você não precisa definir nenhum limite nem configurar regras. Aprendizado de máquina e algoritmos de mineração de dados são usados para detectar padrões anormais.

Estamos ansiosos para receber seus comentários. Diga-nos como ele te ajuda, como podemos aprimorar a detecção proativa e quais recursos adicionais você deseja ver adicionados. Você pode fornecer comentários em Enviar um Smiley/Rosto Triste no portal ou enviar um email para AppInsightsML@microsoft.com.

## Sobre o alerta proativo

* *Por que eu recebi esse email?*
 * A detecção proativa analisou a telemetria que seu aplicativo enviou ao Application Insights e detectou um problema de desempenho em seu aplicativo. 
* *A notificação significa que, definitivamente, tenho um problema?*
 * Não. É simplesmente uma sugestão sobre algo que talvez você queira examinar mais detalhadamente. 
* *O que devo fazer?*
 * [Examine os dados apresentados](#responding-to-an-alert). Use o Metrics Explorer para examinar o desempenho ao longo do tempo e analisar as métricas adicionais. Use a Pesquisa para filtrar eventos específicos que ajudarão você a identificar a causa raiz. 
* *Então, vocês examinam os meus dados?*
 * Não. O serviço é totalmente automático. Somente você recebe as notificações. Os dados são [privados](app-insights-data-retention-privacy.md).


## O processo de detecção

* *Quais tipos de anomalias são detectadas?*
 * Padrões que você acharia demorado verificar por conta própria. Por exemplo, um baixo desempenho em uma combinação específica de local, hora do dia e plataforma.
* *Todos os dados coletados pelo Application Insights são analisados?*
 * Não no momento. Atualmente, analisamos o tempo de resposta de solicitação, o tempo de resposta da dependência e o tempo de carregamento da página. As análises de métricas adicionais estarão disponíveis em breve. 
* *Posso criar minhas próprias regras de detecção de anomalias?*
 * Ainda não. Mas você pode:
 * [Configurar alertas](app-insights-alerts.md) que informam quando uma métrica excede um limite).
 * [Exportar telemetria](app-insights-export-telemetry.md) para um [banco de dados](app-insights-code-sample-export-sql-stream-analytics.md), [para o PowerBI](app-insights-export-power-bi.md) ou para [outras](app-insights-code-sample-export-telemetry-sql-database.md) ferramentas, em que será possível analisá-la por conta própria.
* *Com que frequência a análise é executada?*
 * As análises são executadas todos os dias a partir da telemetria do dia anterior.
* **Então isso substitui os [alertas de métrica](app-insights-alerts.md)?
 * Não. Nosso compromisso não é detectar todos os comportamentos que você pode considerar como anormais.

## Como investigar problemas gerados pela Detecção Proativa

Abra o relatório de anomalias no email ou na lista de anomalias.

![](./media/app-insights-proactive-detection/03.png)


* **Quando** mostra a hora em que o problema foi detectado.
* **O que** descreve
 * O problema que foi detectado;
 * As características do conjunto de eventos que encontramos que apresentaram o comportamento do problema.
* A tabela compara o conjunto com desempenho insatisfatório com o comportamento médio de todos os outros eventos.

Clique nos links para abrir o Metrics Explorer e a Pesquisa de relatórios relevantes, filtrados com base na hora e propriedades do conjunto de desempenho lento.

Modifique o intervalo de tempo e os filtros para explorar a telemetria.

## Como posso melhorar o desempenho?

As respostas lentas e com falha são uma das maiores frustrações para os usuários do site, como você já deve saber por experiência própria. Então é importante resolver os problemas.

### Triagem

Primeiro, isto é importante? Se uma página sempre for carregada lentamente, mas apenas 1% dos usuários do site tiverem de vê-la, talvez haja coisas mais importantes a considerar. Por outro lado, se apenas 1% dos usuários abrir essa página, mas ela lançar exceções todas as vezes, talvez valha a pena investigar.

Use a instrução de impacto no email como um guia geral, mas lembre-se de que isso não é tudo. Obtenha outras evidências para confirmar.

Considere os parâmetros do problema. Se for dependente da geografia, configure os [testes de disponibilidade](app-insights-monitor-web-app-availability.md) incluindo a região; talvez a área esteja simplesmente enfrentando problemas de rede.

### Diagnosticar carregamentos lentos de página 

Onde está o problema? O servidor responde lentamente, a página é muito longa ou o navegador precisa trabalhar muito para exibi-la?

Abra a folha Métrica de navegadores. A [exibição segmentada do tempo de carregamento da página do navegador](app-insights-javascript.md#explore-your-data) mostra onde o tempo está sendo gasto.

* Se o **Tempo de Envio da Solicitação** for alto, o servidor estará respondendo lentamente ou a solicitação será uma publicação com uma grande quantidade de dados. Examine as [métricas de desempenho](app-insights-web-monitor-performance.md#metrics) para investigar os tempos de resposta. 
* Configure o [rastreamento de dependências](app-insights-dependencies.md) para verificar se a lentidão ocorre devido a serviços externos ou ao seu banco de dados.
* Se **Recebendo Resposta** for predominante, sua página e seus componentes dependentes, JavaScript, CSS, imagens e assim por diante (mas não os dados carregados de forma assíncrona) serão longos. Configure um [teste de disponibilidade](app-insights-monitor-web-app-availability.md) e não se esqueça de definir a opção para carregar os componentes dependentes. Ao obter alguns resultados, abra os detalhes de um resultado e amplie-os para ver os tempos de carregamento de arquivos diferentes.
* Um **Tempo de Processamento do Cliente** alto sugere que os scripts estão sendo executados lentamente. Se o motivo não for óbvio, considere a adição de um código de tempo e envie os tempos em chamadas trackMetric.

### Aprimorar páginas lentas

Há muitos conselhos na Web sobre como melhorar os tempos de resposta de seu servidor e de carregamento da página, portanto não os repetiremos aqui. Veja algumas dicas as quais você provavelmente já conhece, apenas para ajudá-lo a pensar:

* Carregamento lento devido a arquivos grandes: carregue os scripts e outros componentes de forma assíncrona. Use o agrupamento de script. Divida a página principal em widgets que carregam os dados separadamente. Não envie o antigo HTML simples para tabelas longas: use um script para solicitar os dados como JSON ou outro formato compacto e preencha a tabela. Há estruturas excelentes para ajudar você com tudo isso. (Que também envolvem scripts grandes, é claro).
* Diminuir as dependências do servidor: considere os locais geográficos de seus componentes. Por exemplo, se você estiver usando o Azure, verifique se o servidor Web e o banco de dados estão na mesma região. As consultas recuperam mais informações do que o necessário? O armazenamento em cache ou o envio em lote ajudaria?
* Problemas de capacidade: examine as métricas do servidor relacionadas aos tempos de resposta e contagens de solicitação. Se os tempos de resposta apresentarem picos desproporcionais, com picos nas contagens de solicitação, é provável que seus servidores estejam alongados. 


## Emails de notificação

* *É necessário assinar este serviço para receber notificações?*
 * Não. Nosso bot periodicamente pesquisa os dados de todos os usuários do Application Insights e envia notificações se detectar problemas.
* *Posso cancelar a assinatura ou ter as notificações enviadas para meus colegas em vez disso?*
 * Clique no link de cancelar assinatura que está no alerta ou no email. 
 
    No momento, elas são enviadas para aqueles com [acesso de gravação ao recurso Application Insights](app-insights-resources-roles-access-control.md).

    Você também pode editar as configurações da lista de destinatários na folha de Detecção Proativa.
* *Não quero ser inundado com essas mensagens.*
 * Elas são limitadas a uma por dia com o problema mais relevante que ainda não relatamos. Você não receberá nenhuma mensagem repetida.
* *Se eu não fizer nada, posso receber um lembrete?*
 * Não, você receberá uma mensagem sobre cada problema apenas uma vez. 
* *Perdi o email. Onde posso encontrar as notificações no portal?*
 * Na visão geral do Application Insights do seu aplicativo, clique no bloco **Detecção Proativa**. Lá, você poderá encontrar todas as notificações de backup de até sete dias anteriores.


## Artigos relacionados

* [Detectar, realizar a triagem e diagnosticar](app-insights-detect-triage-diagnose.md)
* [Definir alertas de métricas](app-insights-alerts.md)
* [Metrics explorer](app-insights-metrics-explorer.md)
* [Gerenciador de pesquisas](app-insights-diagnostic-search.md)
 

<!---HONumber=AcomDC_0121_2016-->
