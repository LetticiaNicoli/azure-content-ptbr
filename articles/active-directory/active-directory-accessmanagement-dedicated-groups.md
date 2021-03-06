<properties
	pageTitle="Grupos dedicados do Active Directory do Azure | Microsoft Azure"
	description="Visão geral do funcionamento e da criação de grupos dedicados no Active Directory do Azure."
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="stevenpo"
	editor=""
	/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/17/2016"
	ms.author="curtand"/>

# Grupos dedicados no Active Directory do Azure

No Azure Active Directory (AD do Azure), o recurso de grupos dedicados automaticamente cria e preenche a associação para grupos predefinidos do Azure AD. Membros de grupos dedicados não podem ser adicionados ou removidos usando o portal clássico do Azure, cmdlets do Windows PowerShell ou programaticamente.

>[AZURE.NOTE] Grupos dedicados requerem que uma licença do Azure AD Premium seja atribuída ao
>- administrador que gerencia a regra em um grupo
>- todos os usuários selecionados pela regra para serem membros do grupo

**Para habilitar grupos dedicados**

1. No [portal clássico do Azure](https://manage.windowsazure.com), selecione **Active Directory** e abra o diretório da sua organização.

2. Selecione a guia **Grupos** e, em seguida, abra o grupo que deseja editar.

3. Selecione a guia **Configurar** e, em seguida, defina **Habilitar Grupos Dedicados** para **Sim**.

Quando a opção Habilitar Grupos Dedicados é definida como **Sim**, você ainda pode habilitar o diretório para criar automaticamente o grupo dedicado Todos os Usuários definindo a **opção Habilitar o Grupo “Todos os Usuários”** como **Sim**. Você pode também editar o nome desse grupo dedicado digitando o **Nome de Exibição para o campo do Grupo “Todos os Usuários”**.

O grupo Todos os Usuários pode ser usado para atribuir as mesmas permissões a todos os usuários no diretório. Por exemplo, você pode conceder a todos os usuários no seu acesso ao diretório para um aplicativo SaaS atribuindo acesso para o grupo dedicado de Todos os Usuários a esse aplicativo.

O grupo dedicado Todos os Usuários inclui todos os usuários no diretório, inclusive convidados e usuários externos. Se você precisar de um grupo que exclua usuários externos, pode fazer isso criando um grupo com uma regra dinâmica baseada em atributos como a seguinte:

				(user.userPrincipalName -notContains "#EXT#@")

Para um grupo que exclui todos os Convidados, use uma regra como a seguinte:

				(user.userType -ne "Guest")

Para saber mais sobre como criar regras *avançadas* (regras que podem conter várias comparações) para a associação dinâmica de grupo, veja [Uso de atributos para criar regras avançadas](active-directory-accessmanagement-groups-with-advanced-rules.md).


Esses artigos fornecem mais informações sobre o Active Directory do Azure.

* [Gerenciamento de acesso a recursos com grupos do Active Directory do Azure](active-directory-manage-groups.md)
* [Índice de artigos para Gerenciamento de Aplicativos no Active Directory do Azure](active-directory-apps-index.md)
* [O que é o Active Directory do Azure?](active-directory-whatis.md)
* [Integração de suas identidades locais com o Active Directory do Azure](active-directory-aadconnect.md)

<!-----------HONumber=AcomDC_0330_2016-->