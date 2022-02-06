# **PuppetDB**

PuppetDB é um serviço open source para guardar dados gerados pelo Puppet, incluindo, por exemplo, catálogos e factos. Foi concebido para reforçar o uso do Puppet com o aumento da perfomance em mente. Guarda toda a informação assíncronamente, libertando o Puppet Master para compilar mais catálogos em simultâneo.

## **Porquê PuppetDB?**

O principal benefício do uso desta ferramenta é o aumento na perfomance mas tem muitas outras vantagens para oferecer. A PuppetDB possui informação sobre cada node, resources, relações entre os nodes e facts dentro da infraestrutura Puppet. Toda esta informação é facilmente visualizada através de queries específicas, para que seja integrada no workflow de um projeto ou até mesmo apenas por mera curiosidade em visualizar estes dados.  
É recomendável utilizar storeconfigs neste caso, visto que podem dar bastante jeito para fazer com que os nodes interajam entre si através do Puppet, que é uma feature extremamente útil. Num caso em que um node precise de saber o que outro node está a fazer nesse momento, storeconfigs podem ajudar.  

## **Perfomance**

O tempo médio para processar um comando é cerca de 394ms, o que é cerca de 130x mais rápido que o pior caso que storeconfigs mais old-school e 10x mais rápido que casos em que os catálogos já se encontram presentes. A PuppetDB responde também às queries numa média de 65ms.  
Estes números tornam-se incomparáveis com o uso de outras bases de dados. Como foi dito anteriormente, é uma grande vantagem guardar assincronamente os dados de modo a libertar o master em vez de ficar mais tempo a aguardar por storeconfigs.

## **Integridade dos dados**

Esta ferramenta guarda todos os aspetos do catálogo que estariam omitidos em storeconfigs mais antigas, tais como resources não exportados e edges.  
Permite também uma maior segurança em relação a não perder os dados que vão sendo guardados através de uma queue persistente que tenta até 16 vezes lidar com determinado comando, garantindo que a data é de qualidade e entra na base de dados.  
O Puppet irá recusar-se a usar quaisquer catálogos que a PuppetDB não tenha conhecimento ou caso esta não esteja disponível, ou seja, os agentes vão usar catálogos sempre atualizados e que estejam presentes na PuppetDB.  
A comunicação com o Puppet Master acontece através de SSL, autenticado com os mesmos certificados utilizados para conectar o master aos agentes.

## **Conclusão**

Apesar da revisão de todas estas vantagens de utilizar PuppetDB, foi decidido neste projeto a não utilização da ferramenta, visto que não se iria justificar a introdução de uma nova peça com este conjunto de funcionalidades.
