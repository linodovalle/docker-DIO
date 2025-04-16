# DOCKER - UTILIZAÇÃO PRÁTICA NO CENÁRIO DE MICROSSERVIÇOS

- Descrição: O presente texto é um resumo adaptativo sobre a aula de demonstração feita pelo professor [Denilson Bonatti](https://github.com/denilsonbonatti) sobre Docker e Swarm, no BootCamp **Santander - Linux para Iniciantes** da [DIO - Digital Innovation One](https://web.dio.me/).

Imagine que, em um mercado, precisamos fazer uma consulta do preço de uma caixa de leite, o código do produto vai para um servidor com uma requisição de informação. Um mercado pode ter o seu servidor em uma sala (um data center) que dá uma resposta de dados ao computador do caixa.

O problema de se ter um datacenter local é o custo, pois uma empresa pode ter milhares de requisições quase simultâneas e, por isso, precisaria de uma máquina muito potente, além de switches de rede, discos de backup, cabeamento, modem, refrigeramento, segurança física e virtual (firewalls, por exemplo)...

Outro problema é que, se houver algum travamento no servidor local, a empresa provavelmente irá parar suas vendas, demandando bastante tempo para que técnicos possam solucionar. Isso porque, com os custos envolvidos na manutenção desse servidor, dificilmente existirá uma máquina de servidor reserva.

No nosso cenário hipotécnico, se a empresa quiser expandir sua operação e criar mais filiais, dificilmente irá instalar outros datacenters nas outras unidades, pois é inviável. Então, ele poderá manter um datacenter grande em uma das unidades e deixar as demais com uma infraestrutura menor.

Essa "nuvem" privada envolve alguns problemas:

- dificuldade com segurança lógica e física
- custo de mão de obra especializada
- custo de hardware
- custo de energia elétrica
- falta de energia (custo com geradores e nobreaks)
- despesas inesperadas (troca de peças)

Portanto, a nuvem pública é uma solução interessante para suprir todas essas questões.

A Amazon, por exemplo, permite que sejam utilizados alguns servidores online gratuitamente (respeitando um limite de dados).

### MICROSSERVIÇOS

É um conceito novo de arquitetura de software que consiste em construir aplicações desmembrando-as em serviços independentes. Estes serviços se comunica entre si usando APIs e promovem grande agilidade em times de desenvolvimento.

Então, no nosso exemplo, teríamos um serviço para consulta de produtos, um para validar a compra via cartão de crédito, outro para dar baixa no estoque, outro para incluir produto no estoque... o sistema passa a ter várias partes diferentes, ao invés de ser uma só aplicação (monolítica).

A vantagem é que podemos ter grupos específicos trabalhando em serviços diferentes. O Netflix e Spotify, por exemplo, possuem mais de 500 microsserviços.

Quando "quebramos" uma aplicação em vários microsserviços, podemos escalá-los de forma separada. Ou seja, se uma empresa tiver em uma situação em que houver muita consulta de preço e pagamento de cartão, não precisará se preocupar com o serviço de login de usuário, ou com a parte de entrada de estoque, por exemplo. Assim, a atenção é voltada somente para as partes (serviços) que estejam sendo mais utilizadas.

Além disso, os microsserviços não necessariamente precisam ser escritos usando a mesma linguagem de programação. Basta que todos sejam capazes de comunicar com o mesmo padrão para que cada microsserviço possa ser escrito e desenvolvido com linguagem diferente e por equipes diferentes, onde cada serviço poderá ser feito com a linguagem que melhor entregue performance, por exemplo.

### CLUSTER

Um cluster ("grupo" ou "aglomerado", em inglês) consiste em computadores ligados que trabalham em conjundo, de modo que podem ser considerados como um único sistema. Computadores em cluster executam a mesma tarefa, controlado e programado por software. Cada computador presente em um cluster é conhecido como nó (node).

### DOCKER SWARM

Swarm é um recurso do Docker que fornece funcionalidades de orquestração de contêiner, incluindo clustering nativo de hosts do Docker e agendamento de cargas de trabalho de contêineres. Um grupo de hosts do Docker formam um cluster "Swarm".

Podemos agendar cargas de recursos de um sistema, de modo que, se precisarmos que o microsserviço de pagamentos de uma empresa fique mais rápido (caso esteja com muita demanda), nós podemos fazer esse balanceamento de maneira rápida.

Outra vantagem é que se, por exemplo, tivermos um cluster com 10 máquinas e uma delas apresente um problema de hardware, os outros nós (as outras máquinas) podem assumir automaticamente. Assim, o microsserviço que estiver rodando na máquina que apresentou problema é "ativado" em outra máquina.

Enfim, vamos ver como funciona isso na prática. Nós iremos:

- subir um container
- testar o container
- subir um cluster

A exposição da aula será feita a partir de 3 máquinas virtuais Ubuntu (1 Gb Ram e 1 Core de processamento cada) na Amazon.

### ENTENDENDO AS DEFINIÇÕES

No DockerHub (hub.docker.com), se fizermos uma pesquisa por MySQL, veremos toda a configuração que deverá ser passada para o nosso servidor (quais as variáveis de ambiente necessárias, posso especificar um password, uma tabela, um banco de dados, podemos criar um usuário e sua senha...).

E também podemos ver na documentação onde que são armazenadas essas informações (em qual diretório).

Na AWS (instância da Amazon), foi criado um container na primeira máquina virtual ("aws-1"), que possui um ip público (e que podemos acessá-lo por um sistema de gerenciamento de banco de dados).

No Fedora, o professor utilizou o Sequeler para adicionar um banco de dados conectando o ip público do 'aws-1'.

Uma vez conectado, podemos criar um banco de dados:


```
CREATE TABLE dados (
    AlunoID int,
    Nome varchar(50),
    Sobrenome varchar(50),
    Endereco varchar(150),
    Cidade varchar(50),
    Host varchar(50)
);
```

Executando a consulta pelo **Sequeler**, se dermos um...
```
Select * FROM dados
```
...poderemos ver a estrutura do banco pronta, mas ainda sem informações.

Então, se tivéssemos desenvolvendo para esse container, já poderíamos usar esse banco.

Agora precisamos criar um aplicação para testar. Então é necessário entrar via SSH na máquina 'aws-01' para criar uma aplicação em PHP para testar a conexão com o banco e se consegue salvar informação lá dentro.

O microsserviço é criado dentro de '/var/lib/docker/volumes'.

Se dermos um 'ls' nesse diretório teremos as pastas 'app' e 'data'
e os arquivos 'metadata.db' e 'backingFsBlockDev'.

Dentro a pasta 'data' é criada uma pasta '_data' para um backup de dados.

Para criar um microsserviço, entra na pasta 'app', '_data' e, com um editor de código (nano, por exemplo) e criamos um arquivo 'index.php':

``` 
<html>
<head>
<title>Exemplo PHP</title>
</head>
<body>

<?php
ini_set("display_errors", 1);
header('Content-Type: text/html; charset=iso-8859-1');

echo 'Versao Atual do PHP: ' . phpversion() . '<br>';

$servername = "54.234.153.24";
$username = "root";
$password = "Senha123";
$database = "meubanco";

// Criar conexão

$link = new mysqli($servername, $username, $password, $database);

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

$valor_rand1 =  rand(1, 999);
$valor_rand2 = strtoupper(substr(bin2hex(random_bytes(4)), 1));
$host_name = gethostname();


$query = "INSERT INTO dados (AlunoID, Nome, Sobrenome, Endereco, Cidade, Host) VALUES ('$valor_rand1' , '$valor_rand2', '$valor_rand2', '$valor_rand2', '$valor_rand2','$host_name')";


if ($link->query($query) === TRUE) {
  echo "New record created successfully";
} else {
  echo "Error: " . $link->error;
}

?>
</body>
</html>
```

Basicamente, esse arquivo é uma página HTML com um código PHP incorporado, que irá fazer a conexão com o banco de dados criado na AWS.

A conexão será feita por 'mysqli' (apesar de não ser mais indicado) e três variáveis serão utilizadas para gerar valores randômicos: uma para gerar um número de 1 a 999, outra para gerar um nome e outra para pegar o hostname que está sendo utilizado para o acesso.

Com essas informações geradas aleatoriamente, o código fará um INSERT no banco.

Para rodar um código PHP num servidor, precisamos de um servidor Apache e do PHP instalado. Então precisa criar um container e jogar o arquivo index.php dentro do container.

Comando:
```
docker run --name web-server -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7
```

**-name**: dá um nome

**-dt**: deixa executando em segundo plano (para ficar sempre ativo) e para que crie um prompt de comando caso precise acessar o container futuramente

**-p**: define a porta padrão (precisa liberar no firewall da nossa conta AWS)

**alpine-php7**: uma imagem que já contém o apache, o Linux e o php7 para rodar dentro do container o 'index.php'.


Após a execução do comando, nossa máquina na AWS irá no hub do Docker e irá instalar. Após a instalação, o apache e o php estará rodando sobre o alpine Linux.

Com o container rodando, podemos ir no browser e fazer uma requisição no IP. Como usamos a porta padrão 80 do navegadoro, o browser irá exibir o index.php, que faz uma inclusão no banco. Então, a cada acesso ao meu IP por algum navegador gerará dados aleatórios inseridos no banco de dados.

### ESTRESSANDO O CONTAINER (loader.io)

Como saber se precisamos aumentar a infraestrutura? Fazendo um software de stress, que pode ser feito localmente com softwares específicos ou online pelo site [www.loader.io](www.loader.io).

Para testar pelo loader.io, precisaremos criar um arquivo dentro do nosso servidor com um token de verificação. Isso é uma forma do site não ser utilizado para ataques a sistemas, redes ou sites, pois somente aqueles hosts que tiverem o token gerado serão "estressados" pelo site, como uma maneira de indicar para o loader.io que o teste de stress é voluntário.

Então, dentro do container, cria-se o arquivo com o token conforme instruído pelo site.

No loader.io, clica para verificar o token e depois em adicionar um teste. Pode-se configurar a quantidade de clientes por teste, a quantidade de requisições por minuto, o tempo de teste etc.

Gráficos do loader.io mostrarão se o nosso servidor está lento ou não, pois pode ser verificada a demora entre a requisição e a resposta do servidor.

Podemos ver a média de tempo, a quantidade de banda utilizada...

### INICIANDO UM CLUSTER SWARM

Vamos supor que o teste de stress tenha indicado lentidão e insuficiência. Nesse caso, pega-se o microsserviço que está alocado no container e multiplica ele para mais containers em outros servidores, usando o Swarm do Docker.

No bash da AWS, "mata" o primeiro container (nomeado 'web-server') que é onde está a aplicação. Para apagar:
```
docker rm --force web-server
```
Para verificar se o serviço foi eliminado:
```
docker os
```
Para duplicar o container em vários serviços, cria-se um cluster (Docker Swarm):
```
docker swarm init
```
O Docker irá informar que o Swarm foi inicializado, informando ainda como adicionar um "worker" desse swarm:
```
docker swarm join --token SWMTKN-1-4mndsakdas4askporiugjhc43hkjdkvsidfsuy7t5hsdskajkadhs88nkdfbmqvzopx4312zbxbxshqdsbkjasc39 172.31.0.127:2377
```
**OBS.:** O comando tem duas particularidades: 1) ele fornece um IP local da máquina virtual,para acesso entre as duas máquinas virtuais AWS; e 2) a porta utilizada é a 2377, que é a porta específica utilizada pelo Swarm (logo, precisará liberar essa porta no firewall AWS).

Esse comando acima tem que ser executado no outro servidor ('aws-02') que fará parte do Swarm (do cluster).

Então, pelo terminal:
```
ssh -i ./DOCKER.pem ubuntu@54.221.178.23
```
Acessado o servidor, cola o comando passado para Swarm. O Docker da segunda máquina AWS ('aws-02') informará que o servidor foi adicionado em cluster ("This node joined a swarm as a worker"). Como esse segundo servidor será somente um "worker", significa que ele somente poderá receber container para executar o microsserviço. O Swarm criado não poderá ser gerenciado por esse servidor (somente no 'aws-01').

Além disso, podemos adicionar uma terceira máquina virtual. Basta logar nessa máquina pelo SSH como vimos acima e passar o mesmo comando indicado pelo Docker Swarm.

Caso não saibamos qual o IP da máquina "central" do Docker, basta ir no terminal dela e 'ip a'. Ou pode-se ir pela própria AWS.

### CRIANDO UM SERVIÇO NO CLUSTER

Se dermos um 'docker node ls' listamos todos os "nodes" do Swarm. Algo como:
```
root@aws-01:/var/lib/docker/volumes/app/_data# docker node ls
ID                            HOSTNAME  STATUS       AVAILABILITY  MANAGER STATUS  ENGINE VERSION
a9m2v7qzj3ldx4t5nbgcyu6e1w *  aws-01    Ready        Active        Leader          20.10.12
k1rzx28mve4jqsdu5gobnctlyw    aws-02    Ready        Active                        20.10.12
n3tf5ygzjwau98clrdxk0meqsb    aws-03    Ready        Active                        20.10.12
```
Note que apenas a máquina Leader é nomeada como tal, as demais sendo 'workers'. E todas estão prontas para receber containers.

Agora pode-se replicar, por exemplo, 10 containers nas 3 máquinas automaticamente:
```
docker service create --name web-server --replicas 3 -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7
```
**OBS.:** Não foi utilizado 'docker run' porque não queremos executar UM container, mas sim um serviço de containers. Logo, utiliza-se o 'docker service'.

O serviço será replicado em todos os containers. Nesse caso, serão 03 réplicas.

Para sabermos como ficou replicado:
```
docker service ps web-server
```
Retorna:
```
ID               NAME           IMAGE                             NODE   DESIRED STATE  CURRENT STATE		    ERROR	PORTS
d5tryp1xdzfgao   web-server.1   webdevops/php-apache:alpine-php7  aws-2  Running        Running 38 seconds ago
w4itshs2qee7ago  web-server.2   webdevops/php-apache:alpine-php7  aws-3  Running        Running 36 seconds ago
kkpkflk3forgrago web-server.3   webdevops/php-apache:alpine-php7  aws-3  Running        Running 37 seconds ago
8rrf8o5j24b2ago  web-server.4   webdevops/php-apache:alpine-php7  aws-2  Running        Running 38 seconds ago
xwp219ws8d5nago  web-server.5   webdevops/php-apache:alpine-php7  aws-3  Running        Running 35 seconds ago
dmiw0ob2snzoago  web-server.6   webdevops/php-apache:alpine-php7  aws-1  Running        Running 50 seconds ago
zfmk3k8rxbr6ago  web-server.7   webdevops/php-apache:alpine-php7  aws-1  Running        Running 49 seconds ago
49lc45ls6t9oago  web-server.8   webdevops/php-apache:alpine-php7  aws-1  Running        Running 49 seconds ago
kghubl73o4dxago  web-server.9   webdevops/php-apache:alpine-php7  aws-2  Running        Running 38 seconds ago
t2qppy32f2v7ago  web-server.10  webdevops/php-apache:alpine-php7  aws-2  Running        Running 38 seconds ago
```
Com isso, podemos saber onde foram replicados os containers. Pela tabela acima, podemos ver que foram 3 containers no servidor 1 e 3 e 4 containers no servidor 2.

O problema do cluster é que ele replica o container (e depois podemos redistribuir a carga entre os servidores), só que ele não replica o conteúdo, ou seja, o que está salvo no volume, em '/var/lib/docker/volumes/app/_data'.

Se acessarmos o local acima em cada um dos servidores virtuais (máquinas AWS) e dermos um 'ls' veremos que na máquina Leader os arquivos estarão presentes, mas nas máquinas 'worker' não.

Se o cluster for para o ar dessa forma e alguém acessar o servidor 2 ou 3, a aplicação iria apresentar erro, pois não há os arquivos da aplicação nelas.

### REPLICANDO UM VOLUME DENTRO DO CLUSTER

Para corrigir isso, precisaremos replicar o diretório da aplicação do servidor 1 para os demais servidores. Utilizaremos o pacote 'nfs-server' para essa tarefa, que deverá ser instalado na máquina 1 (onde se encontram os arquivos da aplicação).
```
apt-get install nfs-server
```
E nos outros servidores será instalada a versão cliente do 'nfs':
```
apt-get install nfs-common
```
**OBS.:** a versão server é mais demorada para instalação.

Na máquina do servidor 1, precisa editar o arquivo de configuração do NFS Server:
```
nano /etc/exports
```
Adicionar o conteúdo:

```
/var/lib/docker/volumes/app/_data *(rw,sync,subtree_check)
```

**OBS.:** O ideal seria indicar o IP das máquinas que terão acesso à replicação da pasta da aplicação, mas na aula o professor liberou para todas as máquinas do cluster (utilizando para isso "*"). Assim, todas as máquinas poderão gravar (rw), sincronizar (sync) e checkar as subpastas (subtree_check).

Configurado o arquivo 'exports' do '/etc', vamos exportar a pasta:
```
exportfs -ar
```
Pelo comando abaixo, podemos ver o que está sendo compartilhado:
```
showmount -e
```
Agora, em cada máquina cliente devemos criar o mesmo caminho de diretório existente no servidor 1 e montar o diretório exportado lá. Com 'mkdir' crie o caminho '/var/lib/Docker/volumes/app/_data' e entre nele. Então:
```
mount -o v3 172.31.0.127:/var/lib/Docker/volumes/app/_data /var/lib/Docker/volumes/app/_data
```
Esse 'v3' foi usado sem muita explicação, mencionando apenas que nas versões anteriores esse comando do Ubuntu não roda.

Esse processo costuma demorar um pouco, então enquanto uma está sendo feita, pode-se ir fazendo nas outras máquinas virtuais.

Se tivéssemos mais máquinas virtuais para rodar nosso cluster, todas precisariam desse processo. Existem outras soluções que podem otimizar esse processos, mas para fazer via linha de comando como vimos demanda bastante tempo.

Pode-se usar IaC para automatizar uma boa parte dessas tarefas.

### CRIANDO UM PROXY UTILIZANDO O NGINX

Agora precisaremos criar um proxy, pois na hora que vier uma requisição na primeira máquina ('aws-01') precisará replicar para todos os outros containers automaticamente: através de um serviço pago na AWS ou usando um proxy interno no servidor 1, que também será um container que fará essa replicação.
```
cd /
nkdir proxy
cd proxy
```
Então vamos criar um arquivo de configuração desse proxy, pois precisamos informar para ele quais máquinas são pertencentes a ele:
```
nano nginx.conf
```

Conteúdo:

```
http {
   
    upstream all {
        server 172.31.0.37:80;
        server 172.31.0.151:80;
        server 172.31.0.149:80;
    }

    server {
         listen 4500;
         location / {
              proxy_pass http://all/;
         }
    }

}

events { }
```

Pelo arquivo de proxy acima, quando fizermos uma requisição na porta 4500, ele replicará para todos os servidores "upstream" acima configurados. E o cluster irá separar as requisições automaticamente em cada container. Então teremos 3 hosts com vários containers nesse microsserviço. Se precisarmos de mais containers, basta ampliar.

Agora vamos pegar esse arquivo de configuração e mandar para dentro do container do proxy que vamos criar. Para facilitar, criaremos um "Docker File", que é um arquivo de configuração do container onde indicamos a imagem que vamos usar e o que queremos fazer com ela.

Nós vamos usar a imagem do proxy NGINX e vamos informar no Docker File que ele deve pegar o NGINX e colocar o arquivo de configuração dentro dele. Para criar o Docker File:
```
nano dockerfile
```
Conteúdo:
```
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

Indicamos qual a imagem que vamos usar (FROM) e qual o arquivo que vamos copiar (COPY). Assim, o Docker irá no Hub e vai buscar o NGINX. Depois que ele baixar no host, ele vai buscar o arquivo de configuração feito e vai jogar dentro dos containers. Dessa forma, tudo irá já configurado.

Então agora precisamos subir um container com essa configuração que criamos:
```
root@aws-1:/proxy# docker build -t proxy-app .
```
O "." indica o envio de todos os arquivos da pasta '/proxy', que é o local de onde executamos o 'docker build'.

Para conferirmos os envios, usamos:
```
root@aws-1:/proxy# docker image ls
```
Retorna:
```
REPOSITORY              TAG           IMAGE ID       CREATED          SIZE
proxy-app               latest        8a782a07ee1d   12 seconds ago   142MB
mysql                   5.7           0712d5dc1b14   6 days ago       448MB
nginx                   latest        c316d5a335a5   7 days ago       142MB
webdevops/php-apache    alpine-php7   727c1b287b3f   22 months ago    286MB
```
Nessa listagem podemos ver que nessa máquina temos a imagem do Apache, do MySQL, o NGINX e o Proxy-App que acabamos de criar com o "Docker File", já contendo a configuração. Então já podemos usar o comando:
```
docker container run --name my-proxy-app -dti -p 4500:4500 proxy-app
```
Não iremos replicar esse proxy em todos os hosts, mas somente no primeiro.

Então, se dermos um 'docker container ls' podemos verificar que o 'proxy-app' está rodando.

### ESTRESSANDO O CLUSTER

Para testar o proxy e ver se a carga está sendo distribuída pelo cluster, vamos novamente no loader.io, mas agora usaremos a porta 4500 (ao invés da 80). Novamente precisará ser criado um arquivo token em /var/lib/docker/volumes/app/data/_data conforme instruções do site:
```
mkdir loaderio-i123gh1kfgj11ifk1k1hj.txt
```
Dentro do arquivo deverá ser colado o token informado no site. O teste de stress precisa ser no primeiro host. Logo, é no primeiro servidor que você irá salvar o arquivo com o token.

Para sabermos se o cluster está replicando, podemos verificar o log de cada container e checar. Mas observe que, em nosso banco de dados, colocamos uma coluna da nossa tabela para o Host de onde estava chegando a requisição (ou inserção de dados). Então, podemos ir no Sequeler e:
```
SELECT * FROM dados
```
E observar os dados do nosso banco. Pela coluna 'Hosts' podemos ver que a inserção foi feita pelo mesmo host. Mas a partir de determinado momento, a inclusão passa a ser feita de hosts diferentes. Ou seja, o proxy está, automaticamente, fazendo o balanceamento e separação das requisições. Dessa maneira, nenhum servidor ou container será estressado, pois o Docker Cluster irá separar as requisições por cada container existente. Se, no exemplo, temos 3 containers em cada servidor, cada hora a requisição será redirecionada para um deles.

Podemos então aumentar a quantidade de container a medida que for sendo necessário, ou através de IaC ou com a estrutura da própria AWS. Então, quando o processamento em determinado host estiver atingido um patamar predefinido, será criado um cluster de duas máquinas.

Em linhas gerais, essas foram as explicações dadas sobre a demonstração de utilização do Docker feita pelo professor Denilson, da DIO. O texto dá uma ideia das tarefas que devem ser realizadas para o funcionamento da ferramenta. No repositório também encontram-se os arquivos prontos do 'index.php', do proxy etc.

|Escrito por:| [Lino do Valle](https://github.com/linodovalle)|
|------------|-------------------|
|**Conteúdo de**:| [Denilson Bonatti](https://github.com/denilsonbonatti/toshiro-shibakita)|
