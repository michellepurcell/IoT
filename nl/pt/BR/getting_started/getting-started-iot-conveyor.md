---

copyright:
  years: 2017, 2018
lastupdated: "2018-02-24"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}


# Guia 1: Conectando um dispositivo de esteira transportadora  
Crie uma correia transportadora básica com um dispositivo IoT que envia dados de monitoramento ao {{site.data.keyword.iot_full}} no {{site.data.keyword.Bluemix_notm}}.
{:shortdesc}

## Visão geral e objetivo
{: #overview}  
Este guia é o primeiro de uma série para orienta você, passo a passo, pelo processo de conectar dispositivos ao {{site.data.keyword.iot_short_notm}}, monitoramento e agindo com base nos dados do dispositivo, e muito mais. Cada guia se baseia no guia anterior e o código de amostra de cada guia serve como entrada para o próximo. É possível também utilizar esses guias independentes, modificando o código de amostra para atender a suas finalidades.

Neste primeiro guia, nós configuramos uma esteira transportadora conectada e a usamos para enviar dados do IoT para o {{site.data.keyword.iot_short_notm}}.
Dependendo do seu nível de qualificação, você poderá seguir um ou ambos os seguintes caminhos para configurar sua esteira transportadora:
- Caminho A  
Esse caminho ajuda você a começar rapidamente, instalando um aplicativo simulador de esteira transportadora no {{site.data.keyword.Bluemix_notm}}. O aplicativo autorregistra um dispositivo no {{site.data.keyword.iot_short_notm}} e envia automaticamente dados bem formatados para a plataforma. As instruções para esse caminho estão em [Etapa 2A - Usar o aplicativo de amostra de simulador do GitHub](#deploy_app).  
- Caminho B  
Esse caminho é tecnicamente mais desafiador e requer hardware adicional, qualificações em programação Python e o registro manual de seu dispositivo no {{site.data.keyword.iot_short_notm}}. As instruções para esse caminho estão em [Etapa 2B - Construir uma correia transportadora física com um Raspberry Pi e um motor elétrico](#raspberry).

Como parte deste guia, você:
- Criará e implementará uma organização do {{site.data.keyword.iot_short_notm}} usando a CLI do Cloud Foundry.
- Construirá e implementará um dispositivo de esteira transportadora de amostra.
- Conectará o dispositivo de esteira transportadora simulado ao {{site.data.keyword.iot_short_notm}}.

Para iniciar o {{site.data.keyword.iot_short_notm}} usando um dispositivo de IoT diferente, consulte o [Tutorial de introdução](/docs/services/IoT/getting-started.html).
{: tip}

## Pré-requisito
{: #prereqs}

Você precisará das contas e ferramentas a seguir:
* [Conta do {{site.data.keyword.Bluemix_notm}} ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://console.ng.bluemix.net/registration/){: new_window}
* [Cloud Foundry Command Line Interface (cf CLI) ![External link icon](../../../icons/launch-glyph.svg "External link icon")](https://github.com/cloudfoundry/cli#downloads){: new_window}  
Use o cf CLI para implementar apps e serviços no {{site.data.keyword.Bluemix_notm}}. Para obter mais informações, veja a [documentação do Cloud Foundry CLI![External link icon](../../../icons/launch-glyph.svg "External link icon")](https://docs.cloudfoundry.org/cf-cli/){: new_window}
* Opcional: [Git ![External link icon](../../../icons/launch-glyph.svg "External link icon")](https://git-scm.com/downloads){: new_window}  
Se você escolhe usar o Git para fazer download das amostras de código, deve-se ter também uma [conta GitHub.com ![External link icon](../../../icons/launch-glyph.svg "External link icon")](https://github.com){: new_window}. Também é possível fazer download do código como um arquivo compactado sem uma conta GitHub.com.
* Opcional: Um telefone celular no qual executar o aplicativo da web de amostra *Esteira transportadora* para enviar dados do acelerômetro.

## Etapa 1 - Implementar o {{site.data.keyword.iot_short_notm}}  
{: #deploy_watson_iot_platform_service}
As etapas a seguir irão implementar uma instância do serviço do {{site.data.keyword.iot_short_notm}} com o nome `iotp-for-conveyor` em seu ambiente do {{site.data.keyword.Bluemix_notm}}. Se você já tiver uma instância de serviço em execução, será possível usar essa instância com o guia e ignorar esta primeira etapa. Apenas certifique-se de usar o nome do serviço e o espaço do {{site.data.keyword.Bluemix_notm}} corretos quando continuar pelos guias.
{: tip}
1. Na linha de comandos, configure seu terminal de API executando o comando cf api.   
Substitua o valor `API-ENDPOINT` pelo terminal de API para sua região.
   ```
cf api API-ENDPOINT
   ```
Exemplo: `cf api https://api.ng.bluemix.net`
<table>
<tr>
<th>Região</th>
<th>Terminal de API</th>
</tr>
<tr>
<td>Sul dos Estados Unidos</td>
<td>https://api.ng.bluemix.net</td>
</tr>
<tr>
<td>Reino Unido</td>
<td>https://api.eu-gb.bluemix.net</td>
</tr>
 <!--<tr>
 <td>Germany</td>
 <td>https://api.eu-de.bluemix.net</td>
 </tr>-->
</table>
2. Efetue login em sua conta do {{site.data.keyword.Bluemix_notm}}.
  ```
cf login
  ```
Se solicitado, selecione a organização e o espaço em que você deseja implementar a {{site.data.keyword.iot_short_notm}} e o aplicativo de amostra.
3. Implemente o serviço do {{site.data.keyword.iot_short_notm}} no {{site.data.keyword.Bluemix_notm}}.  
   ```
cf create-service iotf-service iotf-service-free YOUR_IOT_PLATFORM_NAME
   ```
Para YOUR_IOT_PLATFORM_NAME, use  * iotp-for-transporyor *.  
Exemplo: `cf create-service iotf-service iotf-service-free iotp-for-conveyor`
3. Crie seu dispositivo de correia transportadora de amostra.  
 - Caminho A: [Etapa 2A - Usar o aplicativo de amostra do simulador por meio do GitHub](#deploy_app).  
 - Caminho B: [Etapa 2B - Construir uma esteira transportadora física com um Raspberry Pi e um motor elétrico](#raspberry).  

## Etapa 2A - Implementar o aplicativo da web de esteira transportadora de amostra  
{: #deploy_app}

O aplicativo de amostra permite simular uma esteira transportadora industrial conectada pelo {{site.data.keyword.Bluemix_notm}}. É possível iniciar e parar a esteira e ajustar a velocidade dela. Cada mudança na esteira é enviada para o {{site.data.keyword.Bluemix_notm}} no formato de mensagem MQTT que é exibida no aplicativo. É possível monitorar o comportamento da esteira usando os cartões do painel padrão.
O aplicativo de amostra é construído usando as bibliotecas do cliente Node.js em: [https://github.com/ibm-watson-iot/iot-nodejs ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://github.com/ibm-watson-iot/iot-nodejs){: new_window}

![Conveyor belt app](images/app_conveyor_belt.png "Conveyor belt app")

1. Use sua ferramenta git favorita para clonar o repositório a seguir:  
https://github.com/ibm-watson-iot/guide-conveyor-simulator
No shell do Git, use o comando a seguir:
  ```bash
git clone https://github.com/ibm-Watson-iot/guide-transporyor-simulator
  ```
2. Na linha de comandos, mude o diretório para o diretório no qual o aplicativo de amostra está localizado.
  ```bash
guia cd-transportador-simulador
  ```
3. No diretório *guide-transporyor-simulator*, envie seu app para o {{site.data.keyword.Bluemix_notm}} e dê a ele um novo nome substituindo *YOUR_APP_NAME* no comando cf push. Use a opção `--no-start` porque você iniciará o app no próximo estágio após ele ser ligado ao {{site.data.keyword.iot_short_notm}}.
**Nota:** a implementação de seu aplicativo pode levar alguns minutos.
   ```bash
cf push YOUR_APP_NAME -- no-start
  ```  
1. No diretório *guide-conveyor-simulator*, ligue seu aplicativo à sua instância do {{site.data.keyword.iot_short_notm}} usando os nomes que você forneceu para cada.
  ```bash
cf bind-service YOUR_APP_NAME iotp-for-conveyor
  ```
Para obter mais informações sobre como ligar aplicativos, consulte [Conectando aplicativos](/docs/services/IoT/platform_authorization.html#bluemix-binding).
2. Inicie seu aplicativo para que a ligação entre em vigor.
  ```bash
cf start YOUR_APP_NAME
  ```
Neste estágio, seu aplicativo da web de amostra é implementado em {{site.data.keyword.Bluemix_notm}}.
Quando a implementação é concluída, uma mensagem é exibida para indicar que seu app está em execução.   
Exemplo:
  ```bash
name:              YOUR_APP_NAME
requested state:   started
instances:         1/1
usage:             64M x 1 instances
routes:            YOUR_APP_NAM.mybluemix.net
last uploaded:     Tue 09 May 09:29:49 EDT 2017
stack:             cflinuxfs2
buildpack:         SDK for Node.js(TM) (ibm-node.js-4.8.0,
                   buildpack-v3.11-20170303-1144)
start command:     ./vendor/initial_startup.rb

     state     since                  cpu    memory         disk            details
#0   running   2017-05-09T13:35:08Z   0.0%   19.6M of 64M   66.2M of 256M
  ```
Para ver o status de implementação e a URL do aplicativo, execute o comando a seguir:
  ```bash
cf apps
  ```
Resolva os problemas de erros no processo de implementação usando o comando `cf logs YOUR_APP_NAME --recent`.
{: tip}
1. Em um navegador, acesse o aplicativo.
Abra esta URL: `https://YOUR_APP_NAME.mybluemix.net`    
Exemplo: `https://conveyorbelt.mybluemix.net/`.
2. Insira um ID e token do seu dispositivo.  
O aplicativo de amostra registra automaticamente um dispositivo do tipo `iot-conveyor-belt` com o ID e o token do dispositivo que você forneceu. Para obter mais informações sobre como registrar dispositivos, consulte [Conectando dispositivos](/docs/services/IoT/iotplatform_task.html#iotplatform_subtask1).
4. Continue com a [Etapa 3 - Consulte dados brutos em {{site.data.keyword.iot_short_notm}}](#see_live_data).

## Etapa 2B - Construa uma esteira transportadora desenvolvida com Raspberry Pi
{: #raspberry}

A solução Raspberry Pi é construída usando as bibliotecas do cliente Python em: [https://github.com/ibm-watson-iot/iot-python ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://github.com/ibm-watson-iot/iot-python){: new_window}

### Diagrama esquemático do circuito

![Diagrama de circuito](images/raspi_circuit.png)

### Conexões necessárias

As conexões a seguir entre o Raspberry Pi e o L298N Dual H Bridge Motor Driver Board são obrigatórias. Conecte:

- Pino 2 do Raspberry Pi a +5v na placa L298N
- Pino 6 do Raspberry Pi ao GND na placa L298N
- Pino 13 do Raspberry Pi ao IN1 da placa L298N
- Pino 15 do Raspberry Pi ao IN2 da placa L298N
- +9v da bateria a +12v na placa L298N
- -9v da bateria ao GND na placa L298N
- O nó +ve do motor ao OUT1 na placa L298N
- O nó -ve do motor ao OUT2 na placa L298N

### Requisitos de hardware

1. [Raspberry Pi(2/3) ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://www.raspberrypi.org/){: new_window} com Raspbian Jessie (8.x).
2. [L298N Dual H Bridge Motor Driver Board ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://tronixlabs.com.au/robotics/motor-controllers/l298n-dual-motor-controller-module-2a-australia/){: new_window}
3. Motor de 9V DC
4. Bateria de 9V

### Requisitos de software Raspberry Pi
- Python 2.7.9 e acima, instalado com o Raspbian Jessie (8.x).
- [Biblioteca Python do Watson IoT ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://github.com/ibm-watson-iot/iot-python){: new_window}
- Opcional: Git.  
Se você não tiver o Git instalado em seu Raspberry Pi, poderá instalá-lo usando o comando a seguir:  
```bash
$ sudo apt-get install git
```

### Etapas detalhadas

1. Abra o terminal ou SSH para o Raspberry Pi.
2. Use sua ferramenta Git favorita para clonar o repositório a seguir no Raspberry Pi:
https://github.com/ibm-watson-iot/guide-conveyor-rasp-pi
No shell Git, use o comando a seguir:
```bash
$ git clone https://github.com/ibm-watson-iot/guide-conveyor-rasp-pi
```
3. Registre o dispositivo no {{site.data.keyword.iot_short_notm}}.  
Para obter mais informações sobre como registrar dispositivos, consulte [Conectando dispositivos](/docs/services/IoT/iotplatform_task.html#iotplatform_subtask1).
	 1. No console do {{site.data.keyword.Bluemix_notm}}, clique em **Ativar** na página de detalhes do serviço do {{site.data.keyword.iot_short_notm}}.
	     O console da web do {{site.data.keyword.iot_short_notm}} é aberto em uma nova guia do navegador na URL a seguir:
	     ```
	     https://ORG_ID.internetofthings.ibmcloud.com/dashboard/#/overview
	     ```
	     Em que ORG_ID é o ID exclusivo de seis caracteres de [sua organização do {{site.data.keyword.iot_short_notm}}](/docs/services/IoT/iotplatform_overview.html#organizations){: new_window}.
	 2. No painel Visão geral, na área de janela de menu, selecione **Dispositivos** e, em seguida, clique em **Incluir dispositivo**.
	 3. Crie um tipo de dispositivo para o dispositivo que você está incluindo.
	     1. Clique em **Criar tipo de dispositivo**.
	     2. Insira o nome do tipo de dispositivo `iotp-for-conveyor` e uma descrição para o tipo de dispositivo.  
	**Dica:** é possível inserir qualquer nome de tipo de dispositivo, mas as outras guias da série esperam dispositivos que usam o tipo de dispositivo `iotp-for-conveyor`. Se você usa um tipo de dispositivo diferente, deve-se modificar as configurações nos outros guias adequadamente.
	     3. Opcional: insira atributos e metadados de tipo de dispositivo.
	 4. Clique em **Avançar** para iniciar o processo de incluir seu dispositivo com o tipo de dispositivo selecionado.
	 5. Insira um ID de dispositivo, por exemplo, `conveyor_belt`.
	 5. Clique em **Avançar** para concluir o processo.
	 6. Forneça um token de autenticação ou aceite um token gerado automaticamente.
	 7. Verifique se as informações de resumo estão corretas e, em seguida, clique em **Incluir** para incluir a conexão.
	 8. Na página de informações sobre o dispositivo, copie e salve os detalhes a seguir:
	     * ID de Organização
	     * Tipo do Dispositivo
	     * ID do dispositivo
	     * Método de Autenticação
	     * Token de autenticação: você precisará dos valores para o ID da organização, o Tipo de dispositivo, o ID do dispositivo e o Token de autenticação para configurar o dispositivo para se conectar ao {{site.data.keyword.iot_short_notm}}.
4. Navegue para a raiz *guide-conveyor-rasp-pi* do repositório clonado.
5. Torne o script de shell executável usando o comando `sudo chmod +x setup.sh`
6. Execute o arquivo *setup.sh* e insira os detalhes copiados da página de informações do dispositivo quando solicitado.
```bash  
./setup.sh
```  

7. Execute o programa deviceClient.  
Ao executar o programa, o Raspberry Pi inicia o motor e ele será executado por até 2 minutes com base nas configurações de parâmetro.
```bash
python deviceClient.py -t 2
```

Enquanto o motor estiver em execução, o programa publicará eventos do tipo de evento `sensorData` que têm a estrutura de carga útil de amostra a seguir:  
```
{
"d": {
  "elapsed": 1, "running": true, "temperature": 35.00, "ay": "0.00", "rpm": "0.6"
}
}
```

8. Continue com a [Etapa 3 - Consultar dados brutos no {{site.data.keyword.iot_short_notm}}](#see_live_data).

## Etapa 3 - Consultar dados brutos no {{site.data.keyword.iot_short_notm}}
{: #see_live_data}
1. Verifique se o dispositivo está registrado com o {{site.data.keyword.iot_short_notm}}.
 1. Efetue login em seu painel do  {{site.data.keyword.Bluemix_notm}}  em: https://bluemix.net
 2. Em [sua lista de serviços ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://bluemix.net/dashboard/services){: new_window}, clique no serviço *iotp-for-conveyor* do {{site.data.keyword.iot_short_notm}}.
 3. Clique em *Ativar* para abrir o painel do {{site.data.keyword.iot_short_notm}} em uma nova guia do navegador.  
Você pode marcar a URL para acesso fácil depois.   
Exemplo:  ` https:// * iot-org-id * .internetofthings.ibmcloud.com `.
 4. No menu, selecione **Dispositivos** e verifique se seu novo dispositivo é exibido.
2. Enviar dados do sensor para a plataforma.   
O dispositivo envia dados para o {{site.data.keyword.iot_short_notm}} quando as
leituras do sensor mudam. É possível simular esse envio de dados parando, iniciando ou mudando a velocidade da esteira transportadora.   
**Caminho A:** se você estiver acessando o app em um navegador móvel, tente agitar seu smartphone para acionar dados do acelerômetro para a esteira transportadora.
{: tip}

## O que vem a seguir
{: @whats_next}  
Continue com o próximo guia ou vá para outro tópico de seu interesse:
- Caminho A: modifique o app de esteira transportadora para se adequar às suas necessidades.  
Para obter detalhes técnicos, veja:
 - [https://github.com/ibm-watson-iot/guide-conveyor-simulator/blob/master/README.md ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://github.com/ibm-watson-iot/guide-conveyor-simulator/blob/master/README.md){: new_window}
 - [Bibliotecas clienmt do Node.js ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://github.com/ibm-watson-iot/iot-nodejs){: new_window}
 
 ** Nota: **  O Bluemix é agora IBM Cloud. Veja o [IBM Cloud Blog](https://www.ibm.com/blogs/bluemix/2017/10/bluemix-is-now-ibm-cloud/){: new_window} ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo") para obter mais detalhes.
 
- Caminho B: Modifique a configuração do Raspberry Pi conforme suas necessidades.  
Para obter detalhes técnicos, veja:
 - [https://github.com/ibm-watson-iot/guide-conveyor-rasp-pi/blob/master/README.md ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo")](https://github.com/ibm-watson-iot/guide-conveyor-rasp-pi/blob/master/README.md){: new_window}


** Nota: **  O Bluemix é agora IBM Cloud. Veja o [IBM Cloud Blog](https://www.ibm.com/blogs/bluemix/2017/10/bluemix-is-now-ibm-cloud/){: new_window} ![Ícone de link externo](../../../icons/launch-glyph.svg "Ícone de link externo") para obter mais detalhes.

- [ Guia 2: Monitorando os dados do dispositivo ](getting-started-iot-monitoring.html)  
Agora que você conectou um ou mais dispositivos e começou a fazer bom uso dos dados do dispositivo, é hora de começar a monitorar uma coleção de dispositivos.
- [Guia 3: Simulando um grande número de dispositivos](getting-started-iot-large-scale-simulation.html)  
O aplicativo de amostra de esteira transportadora no caminho A permite que você simule
manualmente um ou mais dispositivos de esteira transportadora. Este guia permite configurar um ambiente simulado que tem um grande número de dispositivos.
- [Saiba mais sobre a {{site.data.keyword.iot_short_notm}}](/docs/services/IoT/iotplatform_overview.html)
- [Saiba mais sobre APIs do {{site.data.keyword.iot_short_notm}}](/docs/services/IoT/reference/api.html)
