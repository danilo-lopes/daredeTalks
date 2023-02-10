## Conteúdo

* Algo que provavelmente você não sabia sobre um container (ephemeral);
* A sacada do Docker sobre armazenamento persistente;
* Abstração de volumes/storage no Kubernetes & CSI;
* Demo.

## Algo que provavelmente você não sabia sobre um container (ephemeral)

Por padrão, todos os arquivos criados em um container são armazenados em uma layer que permite escrita de dados, e isso significa que:

1. Esse dado é efêmero;
2. Dificil de mover esse dado para outro lugar.

### Porque um container é efêmero?

![image](https://blog.container-solutions.com/hs-fs/hubfs/understanding_volumes_in_docker.png?width=600&name=understanding_volumes_in_docker.png)

Um Docker image é feita de camadas read only, quando estartamos um container o CRI pega essa imagem e coloca uma camada read/write no seu topo, quando a aplicação altera um arquivo o arquivo é copiado da layer adjacente e adicionado na camada atual. A versão desse arquivo da camada de read/write não substitui o arquivo da camada adjacente, ele ainda existe lá, porém, quando o container é reinicializado ou recriado, essa escrita é perdida.

Com a necessidade de persistência de dados que o Docker veio com conceito de volume. Que são arquivos ou diretórios no host hospedeiro/ou externo que são disponibilizados para o container. Esse volume é segregado do filesystem do container.

## Abstração de volumes/storage no Kubernetes & CSI:

![image](https://support.huaweicloud.com/intl/en-us/basics-cce/en-us_image_0261235726.png)

O Kubernetes tem duas API's responsáveis pelo gerenciamento de storage. o PV, do tipo `PersistentVolume` na versão `/v1` e o tipo `PersistentVolumeClaim`, versão `/v1`.

O PV é um pedaço de storage provisionado pelo adminstrador ou dinâmicamente por um `StorageClass`. Isso é um resource pro cluster, igual como um `Node` é pro Kubernetes. Tem o ciclo de vida independente do POD que usa esse PV. A API provisiona o storage para o usuário dependendo do driver sendo usado ou usando o no próprio in-tree(código fonte do Kubernetes), como o NFS, EBS CSI Driver, EFS, iSCSI.

O PVC é uma requisição de storage feito por um usuário. Ele é parecido com o POD, um POD consome recurso do `Node`, PVC consome recurso do PV. O POD pode requisitar uso de CPU/memoria especifico para o node, o PVC pode requisitar modos de acesso a esse disco (ReadWriteOnce ou ReadWriteMany por exemplo).

O Kubernetes tem a intenção de prover disco como serviço usando `StorageClass`. Quando o usuário requisita um disco o Kubernetes provisiona e entrega ao usuário. A intenção é ter um `StorageClass` para tipos de disco diferentes, por exemplo:

* Com alto IOPS
* Com baixo IOPS
* EBS GP3
* EBS GP2
* EFS

## Container Storage Interface

Como vimos, o Kubernetes possui uma boa capacidade para lidar com storage/volume para containers, porém, como sabemos, no mercado existem diversas tecnologias de storage e manter uma compatibilidade com todos no seu código fonte se tornou inviável e muitas vezes, impossível. Então o CSI foi desenvolvido em cima de um padrão para criação de volume/disco para containers. Com isso os providers de storage podem desenvolver seus drivers e usar como add-on no cluster. Existem diversos drivers, como AWS EBS CSI, AWS EFS CSI, GPC Compute Persistent Disk CSI, GCP Filestore CSI e muitos outros.

![image](https://pic4.zhimg.com/80/v2-60b096d2f866e4ecdcfa2e6218d0e0db_720w.webp)

## Demo

* Instalação de um CSI Driver;
* Provisionamento de um Storage Class;
* Provisionamento de um PVC e disponibilizar para um container;
* Tirar snapshot desse volume;
* Provisionar um novo PVC a partir desse snapshot;
* Validar as configurações.
