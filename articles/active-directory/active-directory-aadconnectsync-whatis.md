<properties
	pageTitle="Sincronização do Azure AD Connect: compreender e personalizar a sincronização | Microsoft Azure"
	description="Explica como funciona a sincronização do Azure AD Connect e como personalizá-lo."
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/18/2016"
	ms.author="markusvi;andkjell"/>


# Sincronização do Azure AD Connect: compreender e personalizar a sincronização
Os Azure Active Directory Connect Synchronization Services (Azure AD Connect Sync) é um componente principal do Azure AD Connect que cuida de todas as operações relacionadas à sincronização de dados de identidade entre o ambiente local e o AD do Azure na nuvem. O serviço de sincronização do Azure AD Connect é o sucessor do DirSync, do AD do Azure Sync e do Forefront Identity Manager com o Azure Active Directory Connector configurado.

Este tópico é a base da **sincronização do Azure AD Connect** (também chamada de **mecanismo de sincronização**) e relaciona links para todos os outros tópicos relacionados a ela. Para obter links para o Azure AD Connect, confira [Integrando suas identidades locais ao Active Directory do Azure](active-directory-aadconnect.md).

O serviço de sincronização consiste em dois componentes, o componente local **Sincronização do Azure AD Connect** e o lado do serviço no Azure AD, chamado **Serviço de sincronização do Azure AD Connect **. O serviço é comum para o DirSync, o Azure AD Sync e Azure AD Connect.

## Tópicos da sincronização do Azure AD Connect

| Tópico | O que ele abrange e quando deve ser lido |
| ----- | ----- |
| **Noções básicas sobre a sincronização do Azure AD Connect** |
| [Compreendendo a arquitetura](active-directory-aadconnectsync-understanding-architecture.md) | Para aqueles que ainda não conhecem o mecanismo de sincronização e querem saber mais sobre a arquitetura e os termos usados. |
| [Conceitos técnicos](active-directory-aadconnectsync-technical-concepts.md) | Uma versão abreviada do tópico sobre arquitetura e que explica brevemente os termos usados. |
| [Topologias para o Azure AD Connect](active-directory-aadconnect-topologies.md) | Descreve as diferentes topologias e os cenários com suporte no mecanismo de sincronização. |
| **Configuração personalizada** |
| [Noções básicas sobre a configuração padrão](active-directory-aadconnectsync-understanding-default-configuration.md)| Descreve as regras prontas e a configuração padrão. Também descreve como as regras trabalham juntas para que os cenários prontos funcionem. |
| [Compreendendo usuários e contatos](active-directory-aadconnectsync-understanding-users-and-contacts.md) | Continua do tópico anterior e descreve como a configuração para usuários e contatos funciona em conjunto, especialmente em um ambiente de várias florestas. |
| [Noções básicas sobre expressões de provisionamento declarativo](active-directory-aadconnectsync-understanding-declarative-provisioning-expressions.md) | Descreve em detalhes o funcionamento do modelo de configuração e a sintaxe para a linguagem de expressão. |
| [Práticas recomendadas para alterar a configuração padrão](active-directory-aadconnectsync-best-practices-changing-default-configuration.md) | Ao conhecer os detalhes dos tópicos acima e precisar fazer alterações na configuração pronta para trabalhar com seu cenário ou com seus requisitos. |
| [Configurar a filtragem](active-directory-aadconnectsync-configure-filtering.md) | Descreve as diferentes opções para limitar quais objetos serão sincronizados com o AD do Azure e o passo a passo de como configurá-los. |
| **Recursos e cenários** |
| [Impedir exclusões acidentais](active-directory-aadconnectsync-feature-prevent-accidental-deletes.md) | Descreve o recurso para *impedir exclusões acidentais* e como configurá-lo. |
| [Agendador](active-directory-aadconnectsync-feature-scheduler.md) | Descreve o agendador interno que está importando, sincronizando e exportando dados. |
| [Implementar a sincronização de senha](active-directory-aadconnectsync-implement-password-synchronization.md) | Descreve o funcionamento da sincronização de senha, como implementá-la, como operá-la e como solucionar problemas. |
| [Write-back de dispositivo](active-directory-aadconnect-feature-device-writeback.md) | Descreve como o write-back de dispositivo funciona no Azure AD Connect. |
| [Extensões de diretório](active-directory-aadconnectsync-feature-directory-extensions.md) | Descreve como estender o esquema do Azure AD com seus próprios atributos personalizados. |
| **Serviço de Sincronização** |
| [Recursos do serviço de sincronização do Azure AD Connect](active-directory-aadconnectsyncservice-features.md) | Descreve o serviço de sincronização e como alterar as configurações de sincronização no Azure AD. |
| [Resiliência do atributo duplicado](active-directory-aadconnectsyncservice-duplicate-attribute-resiliency.md) | Descreve como habilitar e usar a resiliência dos valores de atributos duplicados **userPrincipalName** e **proxyAddresses**. |
| **Operações e interface do usuário** |
| [Synchronization Service Manager](active-directory-aadconnectsync-service-manager-ui.md) | Descreve a interface do usuário do Synchronization Service Manager, incluindo as guias [Operações](active-directory-aadconnectsync-service-manager-ui-operations.md), [Conectores](active-directory-aadconnectsync-service-manager-ui-connectors.md), [Designer de Metaverso](active-directory-aadconnectsync-service-manager-ui-mvdesigner.md) e [Pesquisa de Metaverso](active-directory-aadconnectsync-service-manager-ui-mvsearch.md).|
| [Considerações e tarefas operacionais](active-directory-aadconnectsync-operations.md) | Descreve os problemas operacionais, como a recuperação de desastre. |
| **Mais informações e referências** |
| [Portas](active-directory-aadconnect-ports.md) | Lista as portas que você precisa abrir entre o mecanismo de sincronização e os diretórios locais e o AD do Azure. |
| [Atributos sincronizados com o Active Directory do Azure](active-directory-aadconnectsync-attributes-synchronized.md) | Lista todos os atributos que estão sendo sincronizados entre o AD local e o AD do Azure. |
| [Referência de funções](active-directory-aadconnectsync-functions-reference.md) | Lista todas as funções disponíveis no provisionamento declarativo. |

## Recursos adicionais

* [Integração de suas identidades locais com o Active Directory do Azure](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0420_2016-->