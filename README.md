# Docker | Wordpress e MySQL

# Pré requisitos

Após ter realizado a criação da sua instância, precisamos seguir alguns passos.

## Instalação do Docker

O Docker é responsável por facilitar a criação e administração de ambientes isolados, possibilitando o empacotamento de uma aplicação ou ambiente dentro de um container, se tornando portátil para qualquer outro host que contenha o Docker instalado.

Para conseguir ter acesso ao Docker, é necessário realizar sua instalação. Execute o seguinte comando no seu terminal Linux:

```bash
sudo yum install docker -y
```

Verifique se o sistema está funcionando com o comando `docker --version`

Em seguida, precisamos inicializar o Docker no nosso sistema, com isso, execute:

```bash
sudo service docker start
sudo systemctl enable docker.service
```

Para verificar se o Docker está ativo, execute `systemctl status docker.service`

## Instalação do Docker-compose

O Docker Compose é responsável pela definição e execução de múltiplos containers com base em arquivo de definição, assim, facilitando esse processo de maneira rápida e prática.

Para isso, é preciso realizar sua instalação a partir do comando:

```bash
curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

E para testar, execute:

```bash
docker-compose version
```

## Instalando e configurando o git

Para realizar o versionamento da atividade, usaremos o git. Para isso, basta executar:

```bash
sudo yum install git
```

Para testar, execute:

```bash
git --version
```

Para configurarmos o git, basta executar:

```bash
git config --global user.name "Fulano de Tal"
git config --global user.email fulanodetal@exemplo.br
```

Assim que terminar com a configuração, podemos visualizar executando `git config --list`

## Montagem do NFS

Para conseguir ter acesso ao NFS, deve-se obter o DNS ou o IP de onde será montado nosso NFS. Você deve criar uma pasta a qual deseja realizar a montagem para o NFS entregue. 

Após a criação da pasta que será nosso ponto de montagem e o DNS/IP do nosso NFS, execute o seguinte comando:

```bash
sudo mount -t nfs -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport [DNS-ou-IP]:/ /[pasta-criada-para-montagem]
```

Para saber se o ponto de montagem foi realizado corretamente, basta executar o comando `df` no seu terminal.

### Persistindo o NFS no sistema

Ao montarmos um NFS em nosso sistema, ele poderá ser desconectado a partir do momento que desligamos nossa máquina. Com isso podemos forçar para que essa montagem seja persistida, para assim não precisar realizar a montagem manualmente .

Para isso basta alterarmos o arquivo fstab para ter esse dado persistido no sistema:

1. Abra o arquivo **/etc/fstab** em um editor;
2. Adicione a seguinte linha dentro do arquivo:

```bash
[IP-ou-DNS]:/ /[pasta-para-montagem] nfs nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport 0 0
```

# Criando um arquivo docker-compose.yml

O arquivo docker-compose.yml é onde declaramos nossas instruções e o estado que cada container deve ser criado e operado bem como a comunicação entre eles.

Para o problema proposto, nesse caso, a criação de um Wordpress e MySQL, iremos usar o seguinte documento:

```bash
version: '3.1'

services:
  db:
    image: mysql:latest
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
			MYSQL_RANDOM_ROOT_PASSWORD: wordpress 
    volumes:
      - mysql:/var/lib/mysql

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    restart: always
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
```

Feito isso, basta executar o comando a seguir para iniciar os containers que especificamos a partir do docker-compose:

```bash
docker-compose up -d
```

Com esse comando, os containers devem estar ativos, e para ver a lista de containers ativos, basta executar `docker ps`

# Configuração da EC2

Para esse problema, foi usado uma instância EC2 AWS, com isso, iremos realizar algumas alterações na criação da nossa EC2.

> Para saber mais sobre EC2 acesse o [link](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html).
> 

## Criando grupo de segurança para o Load Balancer

Após ter realizado o [login](https://aws.amazon.com/pt/premiumsupport/knowledge-center/create-and-activate-aws-account/) na plataforma da AWS, entre na aba de EC2 e siga os seguintes passos:

1. No menu esquerdo, entre em Security Groups;
2. Clique em Criar Grupo de Segurança;
3. Insira o nome do seu grupo de segurança, assim como a VPC que vc quer atrelar;
4. Nas regras de saída, selecione o tipo HTTP (Porta 80) para a origem de qualquer local-IPv4;
5. Adicione tags se achar necessário.

> Após a criação do grupo de segurança para o Load Balancer, crie um outro grupo de segurança para ter acesso a sua EC2 (ou se preferir, altere o default), e coloque o tipo HTTP (porta 80) para ter como origem o grupo de segurança que criamos para o Load Balancer
> 

## Criando um Target Group

Para conseguirmos criar o Load Balancer, precisamos definir um Target Group, para isso, basta segui os seguintes passos:

1. No meu esquerdo, selecione Target Groups e em seguida em Create Target Group;
2. Escolha o tipo de target para Instâncias;
3.  Selecione o nome que deseja para o seu TG e em seguida escolha a VPC padrão;
4. Por fim, selecione a instância que deseja colocar no seu TG e clique em Include as panding below;

> Para saber mais sobre Target groups, acesse o [link](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html).
> 

## Criando um **Application Load Balancer**

Para criarmos o Load Balancer, siga o seguinte passo a passo:

1. No menu esquerdo, acesse Load Balancers;
2. Escolha a opção **Application Load Balancer;**
3. Selecione o nome do sua ALB;
4. O esquema deve ser ser voltado para a Internet;
5. Selecione a VPC e mapeie pela Zona de Disponibilidade da sua preferencia (nesse caso, as zonas 1a) e sua Subnet;
6. Selecione o grupo de segurança que criamos apenas para o Load Balancer;
7. Na parte de Roteamento e Listeners selecione o target group que criamos;
8. Adicione tags se preferir;
9. Finalize criando o Load Balancer.

> Para mais informações sobre Application Load Balancer, acesse o [link](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html).
> 

Após todo o procedimento, você deve ser capaz de acessar sua instancia via LB, utilizando o DNS que é disponibilizado.
