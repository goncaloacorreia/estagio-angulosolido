# **Upgrade de versões do Puppet**

## **Instalação do Puppet 7 de raíz**  

Para utilizar o Puppet, há que fazer uma instalação inicial e um processo de setup dos seus componentes.  
O Puppet está distribuído em vários packages, incluindo _puppet server_, _puppet agent_ e _puppetdb_. O Puppet Server controla toda a informação das configurações de um ou mais agent nodes (equivale ao Puppet Master).  
Esta instalação é dividida em diversos passos:

* Instalação do repositório da plataforma Puppet
* Instalação do Puppet Server
* Instalação de um Puppet Agent

A partir da instalação destes componentes separadamente, podemos começar a personalizar as configurações para cada node e instalar outros componentes à medida que a nossa infraestrutura vai crescendo.  

Os comandos de instalação apresentados mais à frente irão ser apenas do tipo **apt** e funcionais apenas em sistemas **Ubuntu**.

### **Instalação do repositório da plataforma Puppet**

A partir de um login do utilizador em root, fazer o download do package e correr o dpkg em install mode:  
`wget <PACKAGE_URL>`  
`sudo dpkg -i <FILE_NAME>.deb`

Utilizando por exemplo, um repositório focal:  
`wget https://apt.puppet.com/puppet7-release-focal.deb`  
`sudo dpkg -i puppet7-release-focal.deb`

E em seguida dar update à lista, dar update à lista de packages apt.

### **Instalação do Puppet Server**

Antes da instalação, há que rever os sistemas operativos suportados e a versão do java, uma vez que este componente corre numa JVM (Java Virtual Machine). Ao contrário do Puppet Agent, o Puppet Server não é suportado para MacOS, mas sim em Red Hat Enterprise Linux 6, 7, 8, Debian 9 (Stretch), 10 (Buster)
Ubuntu 16.04 (Xenial, amd64 only), 18.04 (Bionic), 20.04 (Focal), SLES 12 SP1 (x86_64).