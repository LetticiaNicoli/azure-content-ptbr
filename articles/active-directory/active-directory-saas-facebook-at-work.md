<properties
	pageTitle="Tutorial: integração do Active Directory do Azure com o Facebook at Work | Microsoft Azure"
	description="Saiba como configurar o logon único entre o Active Directory do Azure e o Facebook at Work."
	services="active-directory"
	documentationCenter=""
	authors="asmalser-msft"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/02/2016"
	ms.author="asmalser"/>


# Tutorial: integração do Active Directory do Azure ao Facebook at Work

O objetivo deste tutorial é mostrar como integrar o Facebook at Work com o Azure AD (Active Directory do Azure). <br>A integração do Facebook at Work com o Azure AD oferece os seguintes benefícios:

- Você pode controlar, no Azure AD, quem tem acesso ao Facebook at Work 
- Você pode provisionar contas automaticamente para usuários que receberam acesso ao Facebook at Work
- Você pode habilitar seus usuários a fazerem logon automaticamente no Facebook at Work (logon único) com suas contas do Azure AD
- Você pode gerenciar suas contas em um local central 

Para conhecer mais detalhadamente a integração de aplicativos de SaaS ao Azure AD, consulte [O que é o acesso a aplicativos e logon único com o Active Directory do Azure](active-directory-appssoaccess-whatis.md).


## Pré-requisitos 

Para configurar a integração do Azure AD ao CS Stars, você precisa dos seguintes itens:

- Uma assinatura do AD do Azure
- Facebook at Work com o logon único habilitado

Para testar as etapas deste tutorial, você deve seguir estas recomendações:

- Não use o ambiente de produção, a menos que seja necessário.
- Se não tiver um ambiente de avaliação do Azure AD, você pode obter uma versão de avaliação de um mês [aqui](https://azure.microsoft.com/pricing/free-trial/). 


## Adicionando Facebook at Work da galeria
Para configurar a integração do Facebook at Work ao Azure AD, você precisa adicionar o Facebook at Work por meio da galeria à sua lista de aplicativos SaaS gerenciados.

**Para adicionar o Facebook at Work por meio da galeria, realize as seguintes etapas:**

1. No **Portal de Gerenciamento do Azure**, no painel navegação à esquerda, clique em **Active Directory**. <br><br>![Active Directory][1]<br>

2. Na lista **Diretório**, selecione o diretório para o qual você deseja habilitar a integração de diretórios.

3. Para abrir o modo de exibição de aplicativos, no modo de exibição de diretório, clique em **Aplicativos** no menu superior. <br><br>![Aplicativos][2]<br>

4. Clique em **Adicionar** na parte inferior da página. <br><br>![Aplicativos][3]<br>

5. Na caixa de diálogo **O que você deseja fazer**, clique em **Adicionar um aplicativo da galeria**.<br><br> ![Aplicativos][4]<br>

6. Na caixa de pesquisa, digite **Facebook at Work**.

7. No painel de resultados, selecione **Facebook at Work** e clique em **Concluir** para adicionar o aplicativo.


### Configuração do logon único do AD do Azure

Esta descreve como permitir que os usuários se autentiquem no Facebook at Work com a respectiva conta do Active Directory do Azure usando federação baseada em protocolo SAML.

**Para configurar o logon único do AD do Azure com o Facebook at Work, realize as seguintes etapas:**

1.	Depois de adicionar o Facebook at Work no portal de gerenciamento do Azure, clique em **Configurar Logon Único**.

2.	Na tela **Configurar URL do aplicativo**, digite a URL onde os usuários entrarão no seu aplictivo Facebook at Work. Este é a URL de locatário do Facebook at Work (Exemplo: https://example.facebook.com/)). Ao terminar, clique em **Avançar**.

3.	Em uma janela de navegador da Web diferente, faça logon no site de sua empresa do Facebook at Work como administrador.

4. Siga as instruções na seguinte URL para configurar o Facebook at Work para usar o AD do Azure como um provedor de identidade: [https://developers.facebook.com/docs/facebook-at-work/authentication/cloud-providers](https://developers.facebook.com/docs/facebook-at-work/authentication/cloud-providers)

5.	Depois de concluído, retorne para as janelas de navegador mostrando o portal de gerenciamento do Azure, clique na caixa de seleção para confirmar que você concluiu o procedimento e clique em **Avançar** e em **Concluir**.


## Provisionamento automático dos usuários do Facebook at Work

O AD do Azure suporta a capacidade de sincronizar automaticamente os detalhes da conta de usuários atribuídos ao Facebook at Work. Essa sincronização automática permite que o Facebook at Work obtenha os dados necessários para autorizar o acesso aos usuários antes que eles tentem entrar pela primeira vez. Ele também desprovisiona os usuários do Facebook at Work quando o acesso foi revogado no AD do Azure.

O provisionamento automático pode ser definido clicando em **Configurar provisionamento de conta** na janela do portal de gerenciamento do Azure.

Para obter detalhes adicionais sobre como configurar o provisionamento automático, consulte [https://developers.facebook.com/docs/facebook-at-work/provisioning/cloud-providers](https://developers.facebook.com/docs/facebook-at-work/provisioning/cloud-providers)


## Atribuir usuários a Facebook at Work

Para que os usuários do AAD provisionados possam ver o Facebook at Work em seu painel de acesso, eles devem receber acesso no portal de gerenciamento do Azure.

**Para atribuir usuários ao Facebook at Work:**

1.	Na página inicial do Facebook at Work no portal de gerenciamento do Azure, clique em **Atribuir contas**.

2.	No menu **Mostrar**, selecione se deseja atribuir um usuário ou um grupo ao Facebook at Work e clique no botão Marca de Seleção.

3.	Na lista resultante, selecione os usuários ou grupo aos quais você deseja atribuir o Facebook at Work.

4.	No rodapé da página, clique no botão **Atribuir**.


## Recursos adicionais

* [Lista de tutoriais sobre como integrar aplicativos SaaS com o Active Directory do Azure](active-directory-saas-tutorial-list.md)
* [O que é o acesso a aplicativos e logon único com o Azure Active Directory?](active-directory-appssoaccess-whatis.md)

<!--Image references-->
[1]: ./media/active-directory-saas-cs-stars-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-cs-stars-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-cs-stars-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-cs-stars-tutorial/tutorial_general_04.png

<!---HONumber=AcomDC_0204_2016-->