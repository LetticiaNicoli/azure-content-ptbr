<properties
   pageTitle="Solucionando problemas de falhas da extensão de VM do Windows | Microsoft Azure"
   description="Saiba mais sobre como solucionar falhas da extensão de VM do Windows no Azure"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="top-support-issue,azure-resource-manager"/>

<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="support-article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure-services"
   ms.date="03/29/2016"
   ms.author="kundanap"/>

# Solucionando problemas de falhas da extensão da VM do Windows no Azure

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-rm-include.md)]modelo de implantação clássico.

[AZURE.INCLUDE [virtual-machines-common-extensions-troubleshoot](../../includes/virtual-machines-common-extensions-troubleshoot.md)]

## Exibindo o status da extensão
Os modelos do Azure Resource Manager podem ser executados no Azure PowerShell. Depois que o modelo for executado, o status da extensão poderá ser exibido no Gerenciador de Recursos do Azure ou nas ferramentas de linha de comando.

Aqui está um exemplo:

Azure Powershell:

      Get-AzureVM -ResourceGroupName $RGName -Name $vmName -Status

Veja o exemplo de saída:

      Extensions:  {
      "ExtensionType": "Microsoft.Compute.CustomScriptExtension",
      "Name": "myCustomScriptExtension",
      "SubStatuses": [
        {
          "Code": "ComponentStatus/StdOut/succeeded",
          "DisplayStatus": "Provisioning succeeded",
          "Level": "Info",
          "Message": "    Directory: C:\\temp\\n\\n\\nMode                LastWriteTime     Length Name
              \\n----                -------------     ------ ----                              \\n-a---          9/1/2015   2:03 AM         11
              test.txt                          \\n\\n",
                      "Time": null
          },
        {
          "Code": "ComponentStatus/StdErr/succeeded",
          "DisplayStatus": "Provisioning succeeded",
          "Level": "Info",
          "Message": "",
          "Time": null
        }
    }
  ]

## Solucionando falhas da extensão

### Executando novamente a extensão na VM

Se estiver executando scripts na VM usando a Extensão de Script Personalizado, às vezes, você poderá se deparar com um erro em que a VM foi criada com êxito, mas o script falhou. Nessas condições, a maneira recomendada de se recuperar desse erro é remover a extensão e executar o modelo novamente. Observação: no futuro, essa funcionalidade será aprimorada para eliminar a necessidade de desinstalar a extensão.


#### Remover a extensão do Azure PowerShell

    Remove-AzureVMExtension -ResourceGroupName $RGName -VMName $vmName -Name "myCustomScriptExtension"

Depois que a extensão tiver sido removida, o modelo poderá ser executado novamente para executar os scripts na VM.

<!---HONumber=AcomDC_0420_2016-->