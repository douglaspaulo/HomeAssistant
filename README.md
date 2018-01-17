# Automação com Home Assistant Hass.io

## Introdução

Este projeto destina-se a instalação do sistema de automação residencial Home Assistant, versão Hassio em uma Raspberry Pi 3 Model B com algumas configurações e customizações iniciais para que você possa automatizar dispositivos e tarefas com base em situações e/ou informações de sensores.

## Automação

Automação é um sistema que emprega processos automáticos que comandam e controlam os mecanismos para seu próprio funcionamento. Esta palavra tem origem no grego autómatos que significa mover-se por si ou que se move sozinho.

## Home Assistant

É uma plataforma open-source de automação residencial feita em Python 3. Ele vai ser como um sistema operacional para a sua casa.


## Instalação do Home Assistant Hass.io

### Hardware necessário:

* Raspberry Pi 3 Model B;
* Cartão MicroSD 4GB (onde será instalado o Home Assistant);
* Adaptador SD para cartão MicroSD (utilizado para instalar o Home Assistant);
* PC com leitor de cartões SD e Windows ou MacOSX (neste projeto utilizaremos o sistema Windows);
* Conectividade WiFi através de um roteador que esteja fixo em sua residência;

### Software necessário:

* Imagem [Hass.io para Raspberry Pi 3](https://github.com/home-assistant/hassio-build/releases/download/1.1/resinos-hassio-1.1-raspberrypi3.img.bz2)
* Software [Etcher](https://etcher.io/) para gravar a imagem em um cartão MicroSD;
* Editor de texto, por exemplo Visual Studio Code.


## Vamos à instalação:

1.	Insira o MicroSD no adaptador SD e insira no leitor de SD do seu computador
2.	Abra o Etcher, selecione a imagem do Hass.io que foi baixada, selecione o cartão SD e clique em “Flash”. Isto gravará a imagem do Hass.io no cartão (leva mais ou menos 6 minutos).
3.	Para configurar o WiFi, quando o processo terminar, remova com segurança o cartão e o insira novamente no leitor, abra o arquivo system-connections/resin-sample com o Visual Studio Code. Modifique o parâmetro ssid informando o SSID da sua rede WiFi e o campo psk informando a senha da sua rede.
4.	Remova com segurança o cartão do computador, retire o MicroSD e insira-o na sua Raspberry Pi 3. Se estiver usando uma conexão a cabo com seu roteador, conecte este nela também.
5.	Conecte sua Raspberry Pi 3 na energia e aguarde a inicialização.
6.	Sua Raspberry Pi 3 irá dar o boot, se conectar à internet e baixar a última versão  do Home Assistant, isto demora em média 20 minutos, vá tomar um café e olhar seu Whatsapp.
7.	O Home Assistant estará disponível através do endereço http://hassio.local:8123.
8.	Acessando o endereço do seu Home Assistant, você verá esta tela, aproveite para dar uma olhada na página de componentes do Home Assistant e se interar com as milhares de possibilidades de automações para sua residência. Aguarde o término da instalação.



Depois criamos um novo usuário, `hass`, ele é quem vai executar o Home Assistant. Essa é uma boa pratica de segurança, para não expor o resto do seu sistema controlando acessos e permissões.


```
sudo adduser --system hass
```

Vamos criar uma pasta para a instalação do Home Assistant e mudamos o dono da pasta para o nosso novo usuário.

```
sudo mkdir /srv/hass
sudo chown hass /srv/hass
```

Vamos logar com o novo usuário e configurar o ambiente do Python (virtualenv) na pasta que acabamos de criar. Essa é uma boa prática para não danificar o ambiente de outros aplicativos que já estão rodando no seu computador com as dependências.

```
sudo su -s /bin/bash hass
virtualenv -p python3 /srv/hass
source /srv/hass/bin/activate
```

Agora sim vamos a instalação do Home Assistant,

```
pip3 install --upgrade homeassistant
```

Por fim vamos executar o Home Assistant,

```
sudo -u hass -H /srv/hass/bin/hass
```

Para iniciar o Home Assistant no boot, sempre que seu computador ligar, vamos precisar criar um serviço para o `systemd`. Mas alguém já fez isso, é só fazer download e instalar o serviço.

```
sudo wget https://raw.githubusercontent.com/egermano/open-home/master/autostart/systemd/home-assistant.service -O /etc/systemd/system/home-assistant@hass.service
```

Como a gente criar um usuário para executar o Home Assistant Vamos precisar fazer uma pequena modificação no serviço. Precisamos subistituir `/usr/bin/hass` por `/srv/hass/bin/hass`. A linha que você vai precisar alterar se parece com isso: `ExecStart=/srv/hass/bin/hass --runner`.

```
sudo nano /etc/systemd/system/home-assistant@hass.service
```

Precisamos reiniciar o nosso serviço no `systemd` para que ele utilize essas novas configurações.

```
sudo systemctl --system daemon-reload
sudo systemctl enable home-assistant@hass
sudo systemctl start home-assistant@hass
```

No futuro, se você precisar atualizar o Home Assistant é só usar esses comandos:

```
sudo su -s /bin/bash hass
source /srv/hass/bin/activate
pip3 install --upgrade homeassistant
```

Arquivos importantes:


- Configuração: `/home/hass/.homeassistant/configuration.yaml`
- Logs: `/home/hass/.homeassistant/home-assistant.log`

Minha configuração completa está disponível [aqui](https://github.com/rodrigopdcouto/iothome/blob/master/homeAssistant/configuration.yaml).

Referências:

- [Installation in Virtualenv](https://home-assistant.io/getting-started/installation-virtualenv/)
- [Autostart Using Systemd](https://home-assistant.io/getting-started/autostart-systemd/)

### Mosquitto MQTT broker

Para instalar a última versão do Mosquitto, precisamos usar o repositório deles.

```
wget http://repo.mosquitto.org/debian/mosquitto-repo.gpg.key
sudo apt-key add mosquitto-repo.gpg.key
```

Aí tornamos o repositório deles disponível para o `apt` e atualizamos ele com as novas informações.

```
cd /etc/apt/sources.list.d/
sudo wget http://repo.mosquitto.org/debian/mosquitto-jessie.list
sudo apt-get update
```

Só depois podemos instalar o mosquitto e o cliente dele para testarmos a instalação.

```
sudo apt-get install mosquitto mosquitto-clients
```

O protocolo MQTT suporta as funcionalidades de autenticação e [ACL](https://en.wikipedia.org/wiki/Access_control_list) para proteger e tornar mais seguro o uso.
Para criar e configurar um usuário nós usamos o aplicativo `mosquitto_passwd`.

Então vamos criar o usuário do Home Assistant.

```
cd /etc/mosquitto/conf.d/
sudo touch pwfile
sudo mosquitto_passwd pwfile ha

```
---Shell Commands---

```
sudo apt-get update
sudo apt-get upgrade
```

```
sudo apt-get install mosquitto
sudo apt-get install mosquitto-clients
```

```
sudo nano /etc/mosquitto/mosquitto.conf
```

```
allow_anonymous false
password_file /etc/mosquitto/pwfile
listener 1883
```

```
sudo mosquitto_passwd -c /etc/mosquitto/pwfile username

```

```
mosquitto_sub -d -u username -P password -t "dev/test"
mosquitto_pub -d -u username -P password -t "dev/test" -m "Hello world"

```

E para configurar as permissões de publishing/subscribing, nos precisamos criar o `aclfile`, que vai configurar para cada usuário as permissões de tópicos que ele tem acesso.

```
cd /etc/mosquitto/conf.d/
sudo touch aclfile
```

Exemplos de ACL:

```
user ha
topic write sala/luz1/switch
topic write sala/luz2/switch
...
topic read sala/luz1/switch
topic read sala/luz2/switch
```

Aqui o usuário `ha` vai poder ler e excrever no tópicos: `sala/luz1/switch` e `sala/luz2/switch`.

Recomendo que você coloque mais um nível de segurança no seu mosquitto, TLS, assim além do usuário e senha e o ACL você coloca ele com um certificado de segurança. [Veja aqui](mosquitto/tls) as instruçõe para fazer isso com o seu.

```
allow_anonymous false
password_file /etc/mosquitto/conf.d/pwfile
acl_file /etc/mosquitto/conf.d/aclfile
listener 8883 (optional)
cafile /etc/mosquitto/certs/ca.crt (optional)
certfile /etc/mosquitto/certs/raspberrypi.crt
keyfile /etc/mosquitto/certs/raspberrypi.key (optional)
```

Para configurar o Home Assistant utilizar o protocolo MQTT você só precisa adicionar essas informações na configuração.

```yaml
mqtt:
  broker: 'localhost' #127.0.0.1
  port: 8883 #1883
  client_id: 'ha'
  username: 'ha'
  password: '[SUA-SENHA]' (optional)
  certificate: '/etc/mosquitto/certs/ca.crt' (optional)
```

Arquivos importantes:

- Configuração: `/etc/mosquitto/conf.d/mosquitto.conf`
- Logs: `/var/log/mosquitto/mosquitto.log`

Referências:

- [Mosquitto Debian repository](https://mosquitto.org/2013/01/mosquitto-debian-repository/).
