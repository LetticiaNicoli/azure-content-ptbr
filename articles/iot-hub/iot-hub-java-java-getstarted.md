<properties
	pageTitle="Introdução ao Hub IoT para Java | Microsoft Azure"
	description="Tutorial de introdução ao Hub IoT do Azure para Java. Use o Hub IoT do Azure e Java com SDKs de IoT do Microsoft Azure para implementar uma solução de Internet das coisas."
	services="iot-hub"
	documentationCenter="java"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="java"
     ms.topic="hero-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="03/22/2016"
     ms.author="dobett"/>

# Introdução ao Hub IoT do Azure para Java

[AZURE.INCLUDE [iot-hub-selector-get-started](../../includes/iot-hub-selector-get-started.md)]

## Introdução

O Hub IoT do Azure é um serviço totalmente gerenciado que permite comunicações bidirecionais confiáveis e seguras entre milhões de dispositivos IoT (Internet das coisas) e um back-end da solução. Um dos maiores desafios enfrentados por projetos IoT é como conectar dispositivos de forma confiável e segura para a solução de back-end. Para enfrentar esse desafio, o hub IoT:

- Oferece, de forma confiável, mensagens em larga escala do dispositivo para a nuvem e da nuvem para o dispositivo.
- Permite proteger as comunicações usando credenciais de segurança e controle de acesso por dispositivo.
- Inclui bibliotecas de dispositivo para as plataformas e idiomas mais populares.

Este tutorial mostra como:

- Use o portal do Azure para criar um hub IoT.
- Crie uma identidade de dispositivo em seu hub IoT.
- Crie um dispositivo simulado que envie telemetria para o back-end de nuvem.

Ao fim deste tutorial, você terá três aplicativos de console do Java:

* **create-device-identity**, que cria uma identidade do dispositivo e uma chave de segurança associada para conectar o dispositivo simulado.
* **read-d2c-messages**, que exibe a telemetria enviada pelo dispositivo simulado.
* **simulated-device**, que conecta o Hub IoT com a identidade do dispositivo criada anteriormente e envia uma mensagem de telemetria a cada segundo usando o protocolo AMQPS.

> [AZURE.NOTE] O artigo [SDKs do hub IoT][lnk-hub-sdks] fornece informações sobre vários SDKs que você pode usar para criar aplicativos executados em dispositivos e no back-end da solução.

Para concluir este tutorial, você precisará do seguinte:

+ Java SE 8. <br/> [Preparar o ambiente de desenvolvimento][lnk-dev-setup] descreve como instalar o Java para este tutorial no Windows ou no Linux.

+ Maven 3. <br/> [Preparar o ambiente de desenvolvimento][lnk-dev-setup] descreve como instalar o Maven para este tutorial no Windows ou no Linux.

+ Uma conta ativa do Azure. <br/>Se você não tiver uma conta, poderá criar uma conta de avaliação gratuita em apenas alguns minutos Para obter detalhes, consulte [Avaliação gratuita do Azure][lnk-free-trial].

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

Como uma etapa final, clique em **Configurações** na folha do Hub IoT, então, em **Mensagens** na folha **Configurações**. Na folha **Mensagens**, anote o **nome compatível com o Hub de Eventos** e o **ponto de extremidade compatível com o Hub de Eventos**. Você precisará desses valores ao criar o aplicativo **read-d2c-messages**.

![][6]

Você criou seu hub IoT e tem o nome de host do Hub IoT, a cadeia de conexão do Hub IoT, o nome compatível com o Hub de Eventos e o ponto de extremidade compatível com o Hub de Eventos necessário para concluir o restante deste tutorial.

## Criar uma identidade do dispositivo

Nesta seção, você criará um aplicativo do console do Java que cria uma nova identidade do dispositivo no registro de identidade em seu hub IoT. Um dispositivo não pode se conectar ao Hub IoT, a menos que ele tenha uma entrada no registro de identidade do dispositivo. Veja a seção **Registro de identidade do dispositivo** do [Guia do Desenvolvedor do Hub IoT][lnk-devguide-identity] para obter mais informações. Quando você executa esse aplicativo de console, ele gera uma ID e uma chave de dispositivo exclusivas com que seu dispositivo poderá se identificar ao enviar mensagens de dispositivo para a nuvem para o Hub IoT.

1. Crie uma nova pasta vazia chamada iot-java-get-started. Na pasta iot-java-get-started, crie um novo projeto Maven denominado **create-device-identity** usando o comando a seguir no prompt de comando. Observe que é um comando único e longo:

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=create-device-identity -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. No prompt de comando, navegue até a nova pasta create-device-identity.

3. Usando um editor de texto, abra o arquivo pom.xml na pasta create-device-identity e adicione a dependência a seguir ao nó **dependencies**. Isso permite que você use o pacote iothub-service-sdk no aplicativo:

    ```
    <dependency>
      <groupId>com.microsoft.azure.iothub-java-client</groupId>
      <artifactId>iothub-java-service-client</artifactId>
      <version>1.0.2</version>
    </dependency>
    ```
    
4. Salve e feche o arquivo pom.xml.

5. Usando um editor de texto, abra o arquivo create-device-identity\\src\\main\\java\\com\\mycompany\\app\\App.java.

6. Adicione as seguintes instruções **import** ao arquivo:

    ```
    import com.microsoft.azure.iot.service.exceptions.IotHubException;
    import com.microsoft.azure.iot.service.sdk.Device;
    import com.microsoft.azure.iot.service.sdk.RegistryManager;

    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. Adicione as seguintes variáveis no nível da classe à classe **App**, substituindo **{seunomehub}** e **{suachavehub}** pelos valores anotados anteriormente:

    ```
    private static final String connectionString = "HostName={yourhubname}.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey={yourhubkey}";
    private static final String deviceId = "javadevice";
    
    ```
    
8. Modifique a assinatura do método **main** para incluir as exceções mostradas abaixo:

    ```
    public static void main( String[] args ) throws IOException, URISyntaxException, Exception
    ```
    
9. Adicione o código a seguir como o corpo do método **main**. Esse código cria um dispositivo denominado *javadevice* no registro de identidade do Hub IoT, caso ele ainda não exista. Ele então exibe a id e a chave do dispositivo que serão necessárias posteriormente:

    ```
    RegistryManager registryManager = RegistryManager.createFromConnectionString(connectionString);

    Device device = Device.createFromId(deviceId, null, null);
    try {
      device = registryManager.addDevice(device);
    } catch (IotHubException iote) {
      try {
        device = registryManager.getDevice(deviceId);
      } catch (IotHubException iotf) {
        iotf.printStackTrace();
      }
    }
    System.out.println("Device id: " + device.getDeviceId());
    System.out.println("Device key: " + device.getPrimaryKey());
    ```

10. Salve e feche o arquivo App.java.

11. Para compilar o aplicativo **create-device-identity** usando o Maven, execute o comando a seguir no prompt de comando na pasta create-device-identity:

    ```
    mvn clean package -DskipTests
    ```

12. Para executar o aplicativo **create-device-identity** usando o Maven, execute o comando a seguir no prompt de comando na pasta create-device-identity:

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

13. Anote a **Id do dispositivo** e a **Chave do dispositivo**. Você precisará delas mais tarde quando criar um aplicativo que se conecte ao Hub IoT como um dispositivo.

> [AZURE.NOTE] O registro de identidade do Hub IoT armazena apenas as identidades de dispositivo para habilitar o acesso seguro ao hub. Ele armazena as IDs e as chaves de dispositivo para usar como credenciais de segurança e um sinalizador habilitado/desabilitado que permite que você desabilite o acesso para um dispositivo individual. Se seu aplicativo precisar armazenar outros metadados específicos do dispositivo, ele deverá usar um repositório específico do aplicativo. Veja o [Guia do Desenvolvedor do Hub IoT][lnk-devguide-identity] para saber mais.

## Receber mensagens do dispositivo para a nuvem

Nesta seção, você cria um aplicativo do console do Java que lê mensagens do dispositivo para a nuvem do Hub IoT. Um hub IoT expõe um ponto de extremidade compatível com os [Hubs de Eventos][lnk-event-hubs-overview] para permitir que você leia mensagens do dispositivo para a nuvem. Para simplificar, este tutorial cria um leitor básico que não é adequado para uma implantação de alta taxa de transferência. O tutorial [Processar mensagens do dispositivo para a nuvem][lnk-process-d2c-tutorial] mostra como processar as mensagens do dispositivo para a nuvem em escala. O tutorial [Introdução aos Hubs de Eventos][lnk-eventhubs-tutorial] oferece informações adicionais sobre como processar as mensagens dos Hubs de Eventos e é aplicável aos pontos de extremidade compatíveis com o Hub de Eventos do Hub IoT.

> [AZURE.NOTE] O ponto de extremidade compatível com os Hubs de Eventos para ler mensagens do dispositivo para a nuvem sempre usa o protocolo AMQPS.

1. Na pasta iot-java-get-started criada na seção *Criar uma identidade de dispositivo*, crie um novo projeto Maven denominado **read-d2c-messages** usando o comando a seguir no prompt de comando. Observe que é um comando único e longo:

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-d2c-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. No prompt de comando, navegue até a nova pasta read-d2c-messages.

3. Usando um editor de texto, abra o arquivo pom.xml na pasta read-d2c-messages e adicione a dependência a seguir ao nó **dependencies**. Isso permite usar o pacote eventhubs-client em seu aplicativo para ler a partir do ponto de extremidade compatível com Hubs de Eventos:

    ```
    <dependency>
      <groupId>com.microsoft.eventhubs.client</groupId>
      <artifactId>eventhubs-client</artifactId>
      <version>1.0</version>
    </dependency>
    ```

4. Salve e feche o arquivo pom.xml.

5. Usando um editor de texto, abra o arquivo read-d2c-messages\\src\\main\\java\\com\\mycompany\\app\\App.java.

6. Adicione as seguintes instruções **import** ao arquivo:

    ```
    import java.io.IOException;
    import com.microsoft.eventhubs.client.Constants;
    import com.microsoft.eventhubs.client.EventHubClient;
    import com.microsoft.eventhubs.client.EventHubEnqueueTimeFilter;
    import com.microsoft.eventhubs.client.EventHubException;
    import com.microsoft.eventhubs.client.EventHubMessage;
    import com.microsoft.eventhubs.client.EventHubReceiver;
    import com.microsoft.eventhubs.client.ConnectionStringBuilder;
    ```

7. Adicione as seguintes variáveis no nível da classe à classe **App**:

    ```
    private static EventHubClient client;
    private static long now = System.currentTimeMillis();
    ```

8. Adicione a classe aninhada a seguir à classe **App**. O aplicativo cria dois threads para executar **MessageReceiver** para ler as mensagens das duas partições no Hub de Eventos:

    ```
    private static class MessageReceiver implements Runnable
    {
        public volatile boolean stopThread = false;
        private String partitionId;
    }
    ```

9. Adicione o seguinte construtor à classe **MessageReceiver**:

    ```
    public MessageReceiver(String partitionId) {
        this.partitionId = partitionId;
    }
    ```

10. Adicione o método **run** a seguir à classe **MessageReceiver**. Esse método cria uma instância de **EventHubReceiver** para ler em uma partição do Hub de Eventos. Ele fará um loop continuamente e exibirá detalhes da mensagem no console até que **stopThread** seja verdadeiro.

    ```
    public void run() {
      try {
        EventHubReceiver receiver = client.getConsumerGroup(null).createReceiver(partitionId, new EventHubEnqueueTimeFilter(now), Constants.DefaultAmqpCredits);
        System.out.println("** Created receiver on partition " + partitionId);
        while (!stopThread) {
          EventHubMessage message = EventHubMessage.parseAmqpMessage(receiver.receive(5000));
          if(message != null) {
            System.out.println("Received: (" + message.getOffset() + " | "
                + message.getSequence() + " | " + message.getEnqueuedTimestamp()
                + ") => " + message.getDataAsString());
          }
        }
        receiver.close();
      }
      catch(EventHubException e) {
        System.out.println("Exception: " + e.getMessage());
      }
    }
    ```

    > [AZURE.NOTE] Esse método usa um filtro ao criar o receptor para que o receptor leia apenas as mensagens enviadas para o Hub IoT após o início da execução do receptor. Isso será útil em um ambiente de teste para que você possa ver o conjunto de mensagens atual, mas em um ambiente de produção, seu código deverá garantir o processamento de todas as mensagens - consulte o tutorial [Como processar as mensagens do dispositivo para a nuvem do Hub IoT][lnk-process-d2c-tutorial] para saber mais.

11. Modifique a assinatura do método **main** para incluir as exceções mostradas abaixo:

    ```
    public static void main( String[] args ) throws IOException
    ```

12. Adicione o código a seguir ao método **main** na classe **App**. Esse código cria uma instância de **EventHubClient** para conectar o ponto de extremidade compatível com o Hub de Eventos no Hub IoT. Em seguida, ele cria dois threads para ler as duas partições. Substitua **{suachavehubiot}**, **{namespacecompatívelcom seuhubeventos}** e **{nomecompatívelcomseuhub eventos}** pelos valores anotados anteriormente. O valor do espaço reservado **{namespacecompatívelcom seuhubeventos}** é proveniente do **ponto de extremidade compatível com o Hub de Eventos**: ele tem o formato **xxxxnamespace** (em outras palavras, remova o prefixo ****sb://** e o sufixo **.servicebus.windows.net** do valor do ponto de extremidade compatível com o Hub de Eventos do portal).

    ```
    String policyName = "iothubowner";
    String policyKey = "{youriothubkey}";
    String namespace = "{youreventhubcompatiblenamespace}";
    String name = "{youreventhubcompatiblename}";
    try {
      ConnectionStringBuilder csb = new ConnectionStringBuilder(policyName, policyKey, namespace);
      client = EventHubClient.create(csb.getConnectionString(), name);
    }
    catch(EventHubException e) {
        System.out.println("Exception: " + e.getMessage());
    }
    
    MessageReceiver mr0 = new MessageReceiver("0");
    MessageReceiver mr1 = new MessageReceiver("1");
    Thread t0 = new Thread(mr0);
    Thread t1 = new Thread(mr1);
    t0.start(); t1.start();

    System.out.println("Press ENTER to exit.");
    System.in.read();
    mr0.stopThread = true;
    mr1.stopThread = true;
    client.close();
    ```

    > [AZURE.NOTE] Esse código pressupõe que você tenha criado seu hub IoT na camada F1 (gratuita). Um hub IoT gratuito tem duas partições chamadas "0" e "1". Se você criou o hub IoT usando uma das outras camadas de preço, deverá ajustar o código para criar uma **MessageReceiver** para cada partição.

13. Salve e feche o arquivo App.java.

14. Para compilar o aplicativo **read-d2c-messages** usando o Maven, execute o comando a seguir no prompt de comando na pasta read-d2c-messages:

    ```
    mvn clean package -DskipTests
    ```

## Criar um aplicativo de dispositivo simulado

Nesta seção, você criará um aplicativo do console do Java que simula um dispositivo que envia mensagens do dispositivo para a nuvem para um hub IoT.

1. Na pasta iot-java-get-started, criada na seção *Criar uma identidade do dispositivo*, crie um novo projeto Maven denominado **simulated-device** usando o comando a seguir no prompt de comando. Observe que é um comando único e longo:

    ```
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. No prompt de comando, navegue até a nova pasta simulated-device.

3. Usando um editor de texto, abra o arquivo pom.xml na pasta simulated-device e adicione as seguintes dependências ao nó **dependencies**. Isso permite que você use o pacote iothub-java-client em seu aplicativo para se comunicar com o hub IoT e para serializar objetos Java para JSON:

    ```
    <dependency>
      <groupId>com.microsoft.azure.iothub-java-client</groupId>
      <artifactId>iothub-java-device-client</artifactId>
      <version>1.0.2</version>
    </dependency>
    <dependency>
      <groupId>com.google.code.gson</groupId>
      <artifactId>gson</artifactId>
      <version>2.3.1</version>
    </dependency>
    ```

4. Salve e feche o arquivo pom.xml.

5. Usando um editor de texto, abra o arquivo simulated-device\\src\\main\\java\\com\\mycompany\\app\\App.java.

6. Adicione as seguintes instruções **import** ao arquivo:

    ```
    import com.microsoft.azure.iothub.DeviceClient;
    import com.microsoft.azure.iothub.IotHubClientProtocol;
    import com.microsoft.azure.iothub.Message;
    import com.microsoft.azure.iothub.IotHubStatusCode;
    import com.microsoft.azure.iothub.IotHubEventCallback;
    import com.microsoft.azure.iothub.IotHubMessageResult;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.security.InvalidKeyException;
    import java.util.Random;
    import javax.naming.SizeLimitExceededException;
    import com.google.gson.Gson;
    ```

7. Adicione as seguintes variáveis no nível da classe à classe **App**, substituindo **{seunomehubiot}** pelo nome do hub Iot e **{suaiddispositivo}** e **{suachavedispositivo}** pelos valores do dispositivo gerados na seção *Criar uma identidade do dispositivo*:

    ```
    private static String connString = "HostName={youriothubname}.azure-devices.net;DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.AMQPS;
    private static boolean stopThread = false;
    ```

    Este aplicativo de exemplo usa a variável **protocol** quando instancia um objeto **DeviceClient**. Você pode usar o protocolo HTTPS ou AMQPS para se comunicar com o Hub IoT.

8. Adicione a classe aninhada **TelemetryDataPoint** a seguir à classe **App** para especificar os dados de telemetria enviados pelo dispositivo ao hub IoT:

    ```
    private static class TelemetryDataPoint {
      public String deviceId;
      public double windSpeed;
        
      public String serialize() {
        Gson gson = new Gson();
        return gson.toJson(this);
      }
    }
    ```

9. Adicione a classe aninhada **EventCallback** a seguir à classe **App** para exibir o status de confirmação que o hub IoT retorna quando processa uma mensagem a partir do dispositivo simulado. Esse método também notifica o thread principal do aplicativo quando a mensagem tiver sido processada:

    ```
    private static class EventCallback implements IotHubEventCallback
    {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to message with status " + status.name());
      
        if (context != null) {
          synchronized (context) {
            context.notify();
          }
        }
      }
    }
    ```

10. Adicione a classe aninhada **MessageSender** a seguir à classe **App**. O método **run** nessa classe gera dados de telemetria de exemplo a serem enviados para o hub IoT e aguarda uma confirmação antes de enviar a próxima mensagem:

    ```
    private static class MessageSender implements Runnable {
      public volatile boolean stopThread = false;

      public void run()  {
        try {
          double avgWindSpeed = 10; // m/s
          Random rand = new Random();
          DeviceClient client;
          client = new DeviceClient(connString, protocol);
          client.open();
        
          while (!stopThread) {
            double currentWindSpeed = avgWindSpeed + rand.nextDouble() * 4 - 2;
            TelemetryDataPoint telemetryDataPoint = new TelemetryDataPoint();
            telemetryDataPoint.deviceId = "myFirstDevice";
            telemetryDataPoint.windSpeed = currentWindSpeed;
      
            String msgStr = telemetryDataPoint.serialize();
            Message msg = new Message(msgStr);
            System.out.println(msgStr);
        
            Object lockobj = new Object();
            EventCallback callback = new EventCallback();
            client.sendEventAsync(msg, callback, lockobj);
    
            synchronized (lockobj) {
              lockobj.wait();
            }
            Thread.sleep(1000);
          }
          client.close();
        } catch (Exception e) {
          e.printStackTrace();
        }
      }
    }
    ```

    Esse método envia uma nova mensagem de dispositivo para a nuvem um segundo depois que o hub IoT confirma a mensagem anterior. A mensagem contém um objeto serializado por JSON com a deviceId e um número gerado aleatoriamente para simular um sensor de velocidade do vento.

11. Substitua o método **main** pelo código a seguir, que cria um thread para enviar mensagens do dispositivo para a nuvem para o hub IoT:

    ```
    public static void main( String[] args ) throws IOException, URISyntaxException {
    
      MessageSender ms0 = new MessageSender();
      Thread t0 = new Thread(ms0);
      t0.start(); 
    
      System.out.println("Press ENTER to exit.");
      System.in.read();
      ms0.stopThread = true;
    }
    ```

12. Salve e feche o arquivo App.java.

13. Para compilar o aplicativo **simulated-device** usando o Maven, execute o comando a seguir no prompt de comando na pasta simulated-device:

    ```
    mvn clean package -DskipTests
    ```

> [AZURE.NOTE] Para simplificar, este tutorial não implementa nenhuma política de repetição. No código de produção, implemente políticas de repetição (como uma retirada exponencial), como sugerido no artigo [Tratamento de falhas transitórias][lnk-transient-faults] do MSDN.

## Executar os aplicativos

Agora você está pronto para executar os aplicativos.

1. Em um prompt de comando na pasta read-d2c, execute o seguinte comando para iniciar o monitoramento de seu hub IoT:

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    ```

    ![][7]

2. Em um prompt de comando na pasta simulated-device, execute o seguinte comando para iniciar o envio de dados de telemetria para seu hub IoT:

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    ```

    ![][8]

3. O bloco **Uso** no [portal do Azure][lnk-portal] mostra o número de mensagens enviadas para o hub:

    ![][43]

## Próximas etapas

Neste tutorial, você configurou um novo hub IoT no portal e depois criou uma identidade de dispositivo no registro de identidade do hub. Você usou essa identidade de dispositivo para habilitar o aplicativo do dispositivo simulado para enviar mensagens entre o dispositivo e a nuvem para o hub e criou um aplicativo que exibe as mensagens recebidas pelo hub. Você pode continuar a explorar os recursos do hub IoT e outros cenários de IoT nestes tutoriais:

- [Enviar mensagens da Nuvem para o Dispositivo com o Hub IoT][lnk-c2d-tutorial] mostra como enviar mensagens para dispositivos e como processar os comentários de entrega produzidos pelo Hub IoT.
- [Processar mensagens do Dispositivo para a Nuvem][lnk-process-d2c-tutorial] mostra como processar de forma confiável a telemetria e as mensagens interativas vindas dos dispositivos.
- [Carregando arquivos de dispositivos][lnk-upload-tutorial] descreve um padrão que usa mensagens da nuvem para o dispositivo para facilitar o carregamento de arquivo dos dispositivos.

<!-- Images. -->
[6]: ./media/iot-hub-java-java-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-java-java-getstarted/runapp1.png
[8]: ./media/iot-hub-java-java-getstarted/runapp2.png
[43]: ./media/iot-hub-csharp-csharp-getstarted/usage.png

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/library/hh680901(v=pandp.50).aspx

[lnk-eventhubs-tutorial]: ../event-hubs/event-hubs-csharp-ephcs-getstarted.md
[lnk-devguide-identity]: iot-hub-devguide.md#identityregistry
[lnk-event-hubs-overview]: ../event-hubs/event-hubs-overview.md

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/java/device/doc/devbox_setup.md
[lnk-c2d-tutorial]: iot-hub-csharp-csharp-c2d.md
[lnk-process-d2c-tutorial]: iot-hub-csharp-csharp-process-d2c.md
[lnk-upload-tutorial]: iot-hub-csharp-csharp-file-upload.md

[lnk-hub-sdks]: iot-hub-sdks-summary.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-portal]: https://portal.azure.com/

<!---HONumber=AcomDC_0511_2016-->