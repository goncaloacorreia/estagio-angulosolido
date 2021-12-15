# **Upgrade do Puppet 3 para Puppet 7**

## **Puppet 3 para Puppet 5**

### **Puppet Agents**

Apesar de terem havido várias mudanças no Puppet desde a versão 3.8, o processo de upgrade dos agentes do Puppet 3 pode ser automatizado ao contrário dos seus servidores.  O módulo _puppet agent_ executa automaticamente as seguintes tarefas:

* Instalação do repositório Puppet, caso não esteja presente
* Instala a versão mais recente do package _puppet agent_, que substitui as versões instaladas do Puppet, Facter, Hiera e MCollective
* Copia os ficheiros SSL do Puppet para a nova localização
* Copia os ficheiros _puppet.conf_ antigos para a nova localização e limpa settings antigos que tenham sido removidos no Puppet 4 ou que necessitem de um reset para os seus valores default
* Garante que os serviços Puppet e MCollective estão a funcionar corretamente

Processo de instalação dos agentes:

1. Instalação do módulo nos Puppet servers:

   * Caso a gestão do código Puppet seja feita manualmente, instalar através da execução de  
   `puppet module install puppetlabs/puppet_agent --environment <ENVIRONMENT>`
   * Caso seja feita a gestão do código através de r10k, adicionar o módulo e as suas dependências ao Puppetfile.
   * Se a gestão for feita de uma outra forma diferente das anteriores, instalar _puppet agent_ como se estivesse a ser feita a instalação de qualquer outro módulo

2. Atribuir a classe _puppet agent_ a um ou mais nodes:

    Qualquer que seja a forma com que os nodes estejam a ser classificados (main manifest, external node classifier, Hiera, etc.), classificà-los com _puppet agent_.  Também é possível configurar o módulo para controlar quais dos serviços se pretendem inicializar.  
    Esta operação deve ser feita cuidadosamente. A atribuição desta classe deve ser feita num ambiente de teste para garantir que vai funcionar como esperado num sistema equivalente ao do ambiente principal.

3. Depois de feito o upgrade, é recomendado fazer as [post-upgrade clean-up tasks](https://puppet.com/docs/puppet/5.5/upgrade_major_post.html).

### **Puppet Server**

