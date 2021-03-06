<properties
   pageTitle="Como exigir a autenticação multifator | Microsoft Azure"
   description="Aprenda a exigir MFA (multi-factor authentication) para identidades com privilégios com a extensão de Privileged Identity Management do Azure Active Directory."
   services="active-directory"
   documentationCenter=""
   authors="kgremban"
   manager="stevenpo"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="04/15/2016"
   ms.author="kgremban"/>

# Como exigir o MFA no Azure AD Privileged Identity Management

É recomendável que você exija o MFA (multi-factor authentication) para todos os administradores e que todos os administradores atuais e candidatos se registrem para o MFA. Isso reduz o risco de um ataque devido a uma senha comprometida.

Você pode exigir que os usuários concluam um desafio de MFA quando entrarem. A postagem do blog [MFA for Office 365 and MFA for Azure](https://blogs.technet.microsoft.com/ad/2014/02/11/mfa-for-office-365-and-mfa-for-azure/) (MFA para Office 365 e MFA para o Azure) compara o que está incluído nas assinaturas do Office e do Azure, com os recursos contidos na oferta do Microsoft Azure Multi-Factor Authentication.

Você também pode exigir que os usuários concluam um desafio de MFA quando ativarem uma função no Azure AD PIM. Dessa forma, se o usuário não concluir um desafio de MFA quando ao entrar, será solicitado pelo PIM que ele faça isso.

## Solicitação de MFA no Privileged Identity Management do Azure AD

Ao gerenciar identidades no PIM como um administrador de segurança, você poderá ver os alertas que recomendam o MFA para contas com privilégios. Clique no alerta de segurança no painel PIM e uma nova folha será aberta com uma lista das contas de administrador que devem exigir o MFA. É possível exigir o MFA selecionando várias funções e clicando no botão **Corrigir** ou você pode clicar nas reticências ao lado das funções individuais e clicar no botão **Corrigir**.

> [AZURE.IMPORTANT] Como as contas da Microsoft (por exemplo, @outlook.com, @live.com ou @hotmail.com) atualmente não têm suporte para se registrar no Azure MFA, os usuários não serão aceitos como administradores temporários para funções altamente privilegiadas. Se os usuários precisarem continuar a gerenciar cargas de trabalho usando uma conta da Microsoft, converta-os em administradores permanentes por enquanto.

Além disso, você pode alterar o requisito de MFA para uma função específica clicando nela na seção Funções do painel PIM. Em seguida, clique em **Configurações** na folha de função e selecione **Habilitar** em multi-factor authentication.

## Como o PIM do Azure AD valida a MFA

Há duas opções de validação de MFA quando um usuário ativa uma função.

A opção mais simples é contar com o Azure MFA para usuários que estão ativando uma função privilegiada. Para fazer isso, primeiro verifique se esses usuários são licenciados, se necessário, e se estão registrados para o Azure MFA. Há mais informações sobre como fazer isso em [Introdução ao Azure Multi-Factor Authentication na nuvem](../multi-factor-authentication/multi-factor-authentication-get-started-cloud.md#assigning-an-azure-mfa-azure-ad-premium-or-enterprise-mobility-license-to-users). É recomendável, mas não obrigatório, configurar o Azure AD para impor o MFA a esses usuários quando eles entram. Isso ocorre porque as verificações de MFA serão feitas pelo próprio PIM do Azure AD.

Alternativamente, se os usuários se autenticarem localmente, você poderá fazer com que o provedor de identidade seja responsável pelo MFA. Por exemplo, se você tiver configurado os Serviços de Federação do AD para exigir a autenticação baseada em cartão inteligente antes de acessar o Azure AD, [Protegendo os recursos de nuvem usando o Azure Multi-Factor Authentication e o AD FS](../multi-factor-authentication/multi-factor-authentication-get-started-adfs-cloud.md) inclui instruções para configurar o AD FS a fim de enviar solicitações ao Azure AD. Quando um usuário tenta ativar uma função, o Azure AD PIM aceita que o MFA já foi validado para o usuário quando receber as declarações apropriadas.


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## Próximas etapas
[AZURE.INCLUDE [active-directory-privileged-identity-management-toc](../../includes/active-directory-privileged-identity-management-toc.md)]

<!---HONumber=AcomDC_0420_2016-->