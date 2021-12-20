# **Instalação do Puppet 7 de raíz**

Para utilizar o Puppet, há que fazer uma instalação inicial e um processo de setup dos seus componentes.  
O Puppet está distribuído em vários packages, incluindo _puppet server_, _puppet agent_ e _puppetdb_. O Puppet Server controla toda a informação das configurações de um ou mais agent nodes (equivale ao Puppet Master).  
Esta instalação é dividida em diversos passos:

* Instalação do repositório da plataforma Puppet
* Instalação do Puppet Server
* Instalação de um Puppet Agent
* Instalação de uma PuppetDB (opcional)

A partir da instalação destes componentes separadamente, podemos começar a personalizar as configurações para cada node e instalar outros componentes à medida que a nossa infraestrutura vai crescendo.  

Os comandos de instalação apresentados mais à frente irão ser apenas do tipo **apt** e funcionais apenas em sistemas **Ubuntu**.

## **Instalação do repositório da plataforma Puppet**

A partir de um login do utilizador em root, fazer o download do package e correr o dpkg em install mode:  
`wget <PACKAGE_URL>`  
`sudo dpkg -i <FILE_NAME>.deb`

Utilizando por exemplo, um repositório focal:  
`wget https://apt.puppet.com/puppet7-release-focal.deb`  
`sudo dpkg -i puppet7-release-focal.deb`

E em seguida dar update à lista de packages apt:  
`sudo apt-get update`

## **Instalação do Puppet Server**

### **Antes de instalar**

Antes da instalação, há que rever os sistemas operativos suportados. Ao contrário do Puppet Agent, o Puppet Server não é suportado para MacOS, mas sim em Red Hat Enterprise Linux 6, 7, 8, Debian 9 (Stretch), 10 (Buster)
Ubuntu 16.04 (Xenial, amd64 only), 18.04 (Bionic), 20.04 (Focal) e SLES 12 SP1 (x86_64).  
Certificar também que o sistema possui pelo menos 2GB de RAM disponíveis.

### **Instalação**

Agora sim, proceder à instalação do package em si através do seguinte comando:  
`sudo apt-get install puppetserver`

Iniciar o serviço Puppet Server, executando:  
`sudo systemctl start puppetserver`

Verificar se o serviço está instalado corretamente:  
`puppetserver -v`

## **Instalação e configuração dos Puppet Agents**

### **Instalação do Agente**

Para instalar um agente basta executar o seguinte comando na consola:  
`sudo apt-get install puppet-agent`  

Em seguida, iniciar o serviço Puppet:  
`sudo /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true`

### **Configuração**

A configuração dos agentes divide-se em 3 etapas:

1. Configurar o PATH para o acesso aos comandos Puppet:  
Os comandos Puppet estão localizados no diretório bin (_/opt/puppetlabs/bin/_). Este diretório não é a variável PATH por default , ou seja, vamos ter que adicionar o diretório bin ao PATH, através do seguinte comando:  
`export PATH=/opt/puppetlabs/bin:$PATH`  
Esta operação também pode ser feita através da alteração do PATH nos ficheiros de configuração _.profile_ ou _.bashrc_.

2. Configuração do server setting  
Esta etapa permitirá definir a setting _server = puppetserver.example.com_ em puppet.conf.
`puppet config set server puppetserver.example.com --section main`  
Além desta operação é possível alterar outras definições como por exemplo serverport, ca_server, ca_port, report_server, report_port.

3. Conectar o agent ao Server primário e validar o certificado  
Depois de adicionado o Server, é necessário conectar o Puppet agent ao Server primário para que este verifique em intervalos regulares o estado do mesmo, retornando o respetivo catálogo e atualizar a configuração, se necessário.

* Para conectar o agente ao Server primário, executar:  
`puppet ssl bootstrap`  
Irá aparecer uma mensagem do tipo:  
`Info: Creating a new RSA SSL key for <agent node>`

* No node do server primário, validar o certificado:  
`sudo puppetserver ca sign --certname <name>`

* No node do agente, executar de novo o agente:  
`puppet ssl bootstrap`

## **Instalação de uma PuppetDB (opcional)**

A instalação da PuppetDB é opcional e tem como vantagem as suas funcionalidades extra das outras bases de dados, tais como queries melhoradas e reports acerca da infraestrutura.  
Existem 3 opções para implementar esta funcionalidade:

* No caso de a instalação da PuppetDB estar a ser feita no mesmo servidor do Puppet Server, associar as classes _puppetdb_ e _puppetdb::master::config_ ao mesmo.

* Caso o pretendido seja executar a PuppetDB no seu próprio servidor com uma instância de PostgreSQL local, associar a classe _puppetdb_ ao mesmo e associar a classe _puppetdb::master::config_ ao PuppetServer.

* Se o objetivo for implementar a PuppetDB e o PostgreSQL em servidores distintos, associar a classe _puppetdb::server_ e a classe _puppetdb::database::postgresql_ a diferentes servidores, enquanto que a classe _puppetdb::master::config_ fica associada ao PuppetServer. Seria também necessário estabelecer uma conexão SSL entre os servidores PostgreSQL e PuppetDB ([documentação](https://forge.puppet.com/modules/puppetlabs/puppetdb?_ga=2.233211230.1188630887.1638789586-2117569960.1637096939#enable-ssl-connections)). Esta configuração utilizará os certificados do Puppet Agent de ambos os servidores para autenticar e encriptar a comunicação entre as bases de dados.  

Por definição, a PuppetDB torna-se acessível apenas através de localhost, se for necessário aceder através de outro meio, há que alterar o parâmetro listen_address na classe_puppetdb_ ou _puppetdb::server_ da seguinte forma:  
`class { 'puppetdb':`  
    `listen_address => 'example.foo.com'`  
`}`  
