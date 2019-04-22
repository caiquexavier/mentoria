# Curso de Docker - Youtube (Canal LINUXtips) - Descomplicando o Docker

### Aula 1
##### O que é um container?
Arquivo binário que contém tudo necessário para uma aplicação rodar. O container não possui um kernel próprio, então utiliza o kernel da máquina host. Em outras palavras, container é uma virtualização de uma aplicação (e não o sistema operacional como um todo).

Os containeres são gerenciados pela engine do docker, que roda em uma camada acima do sistema operacional.


### Aula 2
##### O que é o Docker?
Surgiu da empresa DotCloud, que era uma PaaS (como Heroku) com a ideia de rodar qualquer linguagem. A DotCloud subiu o projeto de containeres para o Github com código open source, tornando-se Docker. Grandes empresas como Google, Facebook, Spotify, Netflix e RedHat passaram a utilizar o Docker e torná-lo cada vez mais conhecido na comunidade.

### Aula 3
Uma imagem do Docker é composta por diversas camadas read-only. Apenas a última camada (mais acima) é read-write. Quando é preciso modificar um arquivo das camadas mais baixas, o Docker faz uma cópia desse arquivo e coloca na camada de read-write.

Além disso, a escrita é feita pela técnica copy-on-write. Ou seja, os arquivos originais são preservados e qualquer alteração é feita em uma cópia do arquivo.


### Aula 4
Docker utiliza alguns componentes do kernel Linux:

- Namespaces: permite fazer isolamento de componentes do SO. Principal responsável por fazer que o container tenha um ambiente isolado (cada container tem seu ip, mount point, etc)
  - PID namespace responsável por isolar os PIDs. Cada container tem sua própria árvore de Process Identificator. Os processos são isolados em cada container, mas também podem ser visualizados pelo host.
  - net namespace responsável por fazer o isolamento das interfaces de rede
  - mnt namespace faz o isolamento do file system. Cada file system de cada container é isolado e não compartilha com outros
  - ipc namespace responsável pelos semáforos e filas de mensagens padrão POSIX
  - uts namesapce responsável pelo isolamento de hostname do container, versão do SO, etc
  - user namespace responsável por fazer o isolamento do usuários no container

- Cgroups
  - isolamento e gerenciamento de recursos para o container. Tais recursos são CPU, memória, IO, etc

- Netfilter
  - módulo do iptables (para roteamento, redirecionamento de portas, etc)


### Aula 5
Instalação do Docker

- Só roda em processador 64b
- Versão do kernel: `uname - r`
- Versão do docker: `docker --version`


### Aula 6
- Comando `docker` é o CLI usado para interagir com o Docker Daemon
- `docker run image-name` (ex.: `docker run ubuntu`) é utilizado para executar um container. Se a imagem não existir na máquina, o Docker faz o download dela diretamente do Docker Hub
- `docker ps` mostra todos os containeres em execução
- `docker ps -a` mostra todos os containeres que já foram executados na máquina
- `docker images` mostra todas as imagens que estão na máquina
- `docker run -it image-name command` (ex.: `docker run -it ubuntu bash`) abre um terminal interativo dentro do container
- `docker run -d image-name` executa o container como um daemon, um processo em background
- Em sessões interativas no container, digite `ctrl+d` para finalizá-lo. Se quiser apenas sair, mas deixar o container rodando, utilize `ctrl+p+q`. Se quiser voltar para o container que está rodando, utilize `docker attach container-id (ou container-name)`
- `docker create image-name` cria, mas não executa um container
- `docker stop container-id (ou container-name)` para um container em execução
- `docker start container-id (ou container-nane)` executa um container parado
- `docker pause container-id (ou container-name)` pausa um container em execução
- `docker unpause container-id (ou container-name)` despaus um container em pausado
- `docker stats container-id (ou container-name)` mostra quanto de recurso um container está usando (cpu, memória, IO, processos)
- `docker top container-id (ou container-name)` mostra os processos rodando no container
- `docker logs container-id (ou container-name)` mostra os logs gerados pelo container
- `docker rm container-id (ou container-name)` remove um container que está parado
- `docker rm -f container-id (ou container-name)` remove um container que está rodando


### Aula 7
Por padrão, o Docker não tem limites para uso de memória e cpu. Quando há muitos containeres rodando sem gerenciamento de memória e cpu, pode haver problemas de performance neles.

`docker inspect container-id (ou container-name)` mostra diversas informações sobre recursos utilizados pelo container, como memória, cpu, portas, ip, etc.

`docker run --name test debian` é o comando que executa um container com um nome específico.

##### Memória
`docker inspect container-id (ou container-name) | grep -i mem` traz as informações de memória do container. O valor `Memory` indica a quantidade de memória, em MB, utilizada pelo container. Quando é 0, quer dizer que não há limites para o uso de memória. Para criar um container limitando a memória em 512MB, é preciso rodar o comando `docker run -it --memory (ou -m) 512m --name test debian`. Para alterar a quantidade de memória em um container que já está rodando, é preciso utilizar o comando `docker update --memory (ou -m) 1024m container-id (ou container-name)`.

##### CPU
Para limitar a utilização de cpu dos containeres, é preciso fazer o cálculo pela proporção. Supondo que haja 3 containeres rodando: o primeiro tem o limite de cpu de 1024, o segundo de 512 e o terceiro de 512 também. Neste caso, o container 1 pode usar 50% dos recursos de cpu, e os containeres 2 e 3 podem usar 25% dos recursos de cpu cada um.

`docker run -it --cpu-shares 1024 --name test debian` cria o container com os recursos de cpu limitados. Para checar utilize `docker inspect test | grep -i cpu`.

Para criar o cenário acima, utilize `docker run -it --cpu-shares 1024 --name container1 debian`, `docker run -it --cpu-shares 512 --name container2 debian` e `docker run -it --cpu-shares 512 --name container3 debian`. E para checar o que foi criado, `docker inspect container1 container2 container3 | grep -i cpushares`.

Para alterar o cpu de containeres já rodando, utilize `docker update --cpu-shares 256 container-id (ou container-name)`.


### Aula 8
##### Volumes
No Docker, volume é uma forma de colocar um file system (um diretório) dentro do container. Este file system está montado no container, mas não faz parte dele de fato. Em outras palavras, é um diretório de compartilhamento entre o host e o container.

Caso o container seja movido de máquina, o volume não vai junto, pois está persistido no host. O mesmo acontece quando um container termina sua execução. Tudo o que for alterado no diretório dentro do container, também será refletido no host.

Para criar um container especificando um volume, utilize `docker run -it -v /volume (nome do diretório) ubuntu bash`. Dentro do container, utilize o comando `df -h` para listar as partições. Para saber onde o volume está persistido no host, utilize `docker inspect -f {{.Mounts}} container-id (ou container-name)`.

Para mapear um diretório específico do host em um volume do container, crie um diretório no host com `mkdir /root/primeiro_dockerfile`. Crie um container apontando para o diretório criado com `docker run -it -v /root/primeiro_dockerfile:/volume (nome do volume) debian`. Isso faz com que o diretório `/root/primeiro_dockerfile` seja montado no container em `/volumes`.

##### Containeres data-only
Containeres data only que não precisam estar em execução, mas eles possuem volumes que são compartilhados com outros containeres. Para criar um container data-only `docker create -v /data --name dbdados debian`

O parâmetro `--volumes-from` importa volumes de outros containeres para utilizá-los em seu próprio container. Como exemplo, crie dois containeres Postgresql que utilizarão o volume `dbdados`.

```
docker run -p 5432:5432 --name pgsql1 --volumes-from dbdados -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker kamui/postgresql

docker run -p 5432:5432 --name pgsql2 --volumes-from dbdados -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker kamui/postgresql
```

- O parâmetro `-p` expõe uma porta do container ou faz bind de uma porta do container para uma porta do host (host:container).
- O parêmetro `-e` são para variáveis de ambiente. São variáves de ambientes passadas para o container.

Ao executar esses dois containeres do Postgres, o diretório dbdados é preenchido com diversos arquivos criados pelo servidor de banco de dados.