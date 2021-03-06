### Instalar por meio do Composer

1. [instalar o Git][install-git]. Observe que no Windows, também será necessário adicionar o Git executável à variável de ambiente PATH. 

2. Crie um arquivo chamado **composer.json** na raiz do seu projeto e adicione o seguinte código a ele:

	```
	{
	    "repositories": [
	        {
	            "type": "pear",
	            "url": "https://pear.php.net"
	        }
	    ],
	    "require": {
	        "pear-pear.php.net/mail_mime" : "*",
	        "pear-pear.php.net/http_request2" : "*",
	        "pear-pear.php.net/mail_mimedecode" : "*",
	        "microsoft/windowsazure": "*"
	    }
	}
	```

3. Baixe o **[composer.phar][composer-phar]** na raiz do seu projeto.

4. Abra um prompt de comando e execute o seguinte comando na raiz do projeto

	```
	php composer.phar install
	```

### Instalar manualmente

Para baixar e instalar as Bibliotecas de Cliente PHP para o Azure manualmente, siga estas etapas:

> [AZURE.NOTE] As Bibliotecas de Cliente PHP para o Azure têm uma dependência de pacotes PEAR [HTTP\_Request2](http://pear.php.net/package/HTTP_Request2), [Mail\_mime](http://pear.php.net/package/Mail_mime) e [Mail\_mimeDecode](http://pear.php.net/package/Mail_mimeDecode). A maneira recomendada de se resolver essas dependências é instalar esses pacotes usando o [gerenciador de pacotes PEAR](http://pear.php.net/manual/en/installation.php).
 
1. Fazer o download de um arquivo. zip que contém as bibliotecas de [GitHub][php-sdk-github]. Como alternativa, divida o repositório e clone-o para sua máquina local. A última opção requer uma conta do GitHub e ter o Git instalado localmente.
	
2. Copie o diretório `WindowsAzure` do arquivo baixado para a estrutura de diretório de aplicativo.

Para obter mais informações sobre como instalar as Bibliotecas de Clientes PHP para o Azure (incluindo informações sobre instalar como um pacote PEAR), consulte [Baixar o SDK do Azure para PHP][download-SDK-PHP].

[php-sdk-github]: http://go.microsoft.com/fwlink/?LinkId=252719
[install-git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[download-SDK-PHP]: ../articles/php-download-sdk.md
[composer-phar]: http://getcomposer.org/composer.phar

<!---HONumber=AcomDC_0406_2016-->