# Instalação e configuração do r10k

Instalação do r10k via Ruby Gems:

```
/opt/puppetlabs/puppet/bin/gem install r10k
```

Configurar o r10k criando a seguinte estrutura de diretórios e o ficheiro `/etc/puppetlabs/r10k/r10k.yaml` assegurando que contém a seguinte informação:

```
# The location to use for storing cached Git repos
:cachedir: '/var/cache/r10k'
# A list of git repositories to create
:sources:
  # This will clone the git repository and instantiate an environment per
  # branch in /etc/puppetlabs/code/environments
  :my-org:
    remote: 'git@github.com:$_Insert GitHub Organization Here_$/$_Insert GitHub Repository That Will Be Used For Your Puppet Code Here_$'
    basedir: '/etc/puppetlabs/code/environments'
```

Configurar também o ficheiro `/etc/puppetlabs/puppet/hiera.yaml` com os seguintes conteúdos:  
```
---
# Hiera 5 Global configuration file

version: 5

defaults:
  datadir: /etc/puppetlabs/code/environments/%{environment}/hieradata/
  data_hash: yaml_data

hierarchy:
  - name: "Common"
    paths:
      - "common.yaml"

```

## Configurar o Repositório para o Puppet Code

Popular o repositório clonando o mesmo localmente e executando cada uma das seguintes ações:

Ter em conta que o environment default do puppet é `production`. É recomendado alterar o git branch default de `master` para `production` de modo a corresponder. Em alternativa, é possível também definir o environment do agente para `master`.

```
mkdir -p {modules,site/profile/manifests,hieradata}
touch hieradata/common.yaml
touch environment.conf
touch Puppetfile
touch site.pp
```

Editar `environment.conf` e garantir que tem os seguintes conteúdos:

```
manifest = site.pp
modulepath = modules:site
```

Editar `site.pp` e garantir que tem os seguintes conteúdos:

```
node agent1.local {
  include ntp
}

class { 'ntp':
}
```

Editar `hieradata/common.yaml e garantir que tem os seguintes conteúdos:
```
---
ntp::servers:
  - 0.europe.pool.ntp.org
  - 1.europe.pool.ntp.org
  - 2.europe.pool.ntp.org
  - 3.europe.pool.ntp.org
```
Editar `Puppetfile` e garantir que tem os seguintes conteúdos:
```
forge 'forge.puppetlabs.com'

# Forge Modules
mod 'puppetlabs-ntp', '9.1.0'
mod 'puppetlabs/stdlib'
```

Garantir que o r10k corre de maneira a aceder ao repositório git. Pode-se testar o acesso ao mesmo usando `sudo git clone yourrepoURL`.
## Resumo
Temos agora as seguintes peças a funcionar:
1. Puppet master
2. Hiera
3. r10k
4. Repositório para o Puppet Code
5. 'profile' inicial com o nome 'base' que irá configurar o NTP nos agentes.
Esta base irá servir para fazer variadas funções. A parte mais interessante é mesmo a capacidade de utilizar Git branches para ajudar a gerir a infraestrutura como parte do ciclo de vida do desenvolvimento de software. Agora, quando for necessário testar um novo profile, podemos fazer o seguinte:
1. Criar um novo branch no repositório
2. Criar código Puppet neste novo branch
3. Push deste novo branch para o repositório
4. Deploy como um novo environment usando: `sudo /opt/puppetlabs/puppet/bin/r10k deploy environment -p`.  
A partir de qualquer agente (incluindo o master), podemos correr o teste a este novo environment, especificando-o na linha de comandos. Por exemplo, se a nova branch tiver o nome `teste`, utilizar o seguinte comando:
```
sudo -i puppet agent -t --environment teste
```
Como foi dito anteriormente, podemos também modificar `/etc/puppetlabs/puppet/puppet.conf` num node e adicionar um environment predefinido:
```
...
[agent]
environment = teste
```
