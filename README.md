# Banco de Dados Oracle com Docker

Esse exemplo explica como utilizar um banco de dados Oracle em containers Docker, é muito recomendado para economizar tempo com setup de ambiente desenvolvimento.

Para mais informações sobre o Oracle visite: [Oracle Database Online Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/index.html).

Para mais informações e detalhes sobre o assunto visite: [Docker Images on Github](https://github.com/oracle/docker-images)

## Como fazer o build da imagem Oracle

**IMPORTANTE:** é importante saber que o repositório não fornece o pacote de instalação do Oracle, para que o build funcione bem é preciso obter o pacote no site da Oracle, correspondente a versão atual desse tutorial.

Os binários podem ser baixados em [Oracle Technology Network](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)

* Obtenha o pacote .zip de instalação no site da Oracle;
* Certifique que o .zip tenha como pasta raiz a "database";
* Certifique que o .zip tenha o nome de linuxx64_12201_database.zip;
* Certifique de copiar o zip dentro da pasta que está o Dockerfile.

### Criando o build da imagem

Esse projeto oferece um exemplo de Dockerfile para o Oracle Database 12c Release 1 (12.1.0.2) Enterprise Edition ou Standard Edition 2

Edição que pode ser escolhida no momento do build.

_Considere como workspace a pasta que está o Dockerfile._

Para criar uma imagem do Oracle Enterprise execute o seguinte script:

    docker build --force-rm=true \
    --no-cache=true \ 
    --build-arg DB_EDITION=ee \
    -t oracle/db:12.2-ee \
    -f Dockerfile .

* ```DB_EDITION=ee``` cria uma imagem do Oracle Enterprise
* ```DB_EDITION=se2``` cria uma imagem do Oracle Standard Edition 2

### Iniciando o container docker

Abaixo temos a descrição completa do comando ```docker run``` que podemos executar:

    docker run --name <container name> \
    -p <host port>:1521 -p <host port>:5500 \
    -e ORACLE_SID=<nome do SID default: ORCLCDB> \
    -e ORACLE_PDB=<nome do PDB default: ORCLPDB1> \
    -e ORACLE_PWD=<senha dos usuários/schmemas> \
    -e INIT_SGA_SIZE=<memória SGA em MB opcional> \
    -e INIT_PGA_SIZE=<memória PGA em MB opcional> \
    -e ORACLE_CHARACTERSET=<caracteres do banco default: AL32UTF8> \
    -v [<volume no host>:]/opt/oracle/oradata \
    oracle/db:12.2-ee

Aqui temos um comando resumido e funcional:

    docker run --name meu-oracle-12 \ 
    -p 1521:1521 \ 
    -p 5500:5500 \
    -e ORACLE_PWD="NzTQYrE$a8Fd" \
    oracle/db:12.2-ee

Caso seja definida uma senha fraca o Oracle pode não aceita-la.

Ou caso tenha esquecido a senha dos usuários ```SYS, SYSTEM e PDBADMIN``` é possível executar o shell de alteração de senha direto no docker:

    docker exec <nome container> /opt/oracle/setPassword.sh <nova senha>

### Conlclusão

Com essa sequencia de passos acima você terá uma imagem compilada e um container rodando do Oracle Database Enterprise e totalmente funcional para desenvolvimento.

### Criação de novo usuário

Em alguns casos é preciso criar um usuário/schema com um determinado nome.

Considere que todos os comandos abaixo estão sendo executados dentro do container docker, utilizando o sqlplus:

    docker exec -it <nome container> /bin/bash

Para acessar o Oracle pelo sqlplus você pode executar os seguinte comando:

    sqlplus / as sysdba

Após conectado ao Oracle vamos descobrir quais os PDB's existem e vamos alterar o contexto para um deles. No exemplo abaixo vamos considerar o PDB ```default=ORCLPDB1```

    SELECT PDB FROM V$SERVICES;
    ALTER SESSION SET CONTAINER=ORCLPDB1;

Alterado para o PDB desejado vamos iniciar com a criação do usuário. No exemplo abaixo vamos também criar uma TABLESPACE própria para o usuário:

    CREATE TABLESPACE MEU_USUARIO_DAT DATAFILE 'MEU_USUARIO_DAT.dat' SIZE 20M AUTOEXTEND ON ONLINE;
    CREATE TEMPORARY TABLESPACE MEU_USUARIO_DAT_TEMP TEMPFILE 'MEU_USUARIO_DAT_TEMP.dbf' SIZE 5M AUTOEXTEND ON;

Logo após criar a TABLESPACE, vamos criar o usuário e definir seus privilégios:

    CREATE USER MEU_USUARIO IDENTIFIED BY admin DEFAULT TABLESPACE MEU_USUARIO_DAT TEMPORARY TABLESPACE MEU_USUARIO_DAT_TEMP;
    GRANT ALL PRIVILEGES to MEU_USUARIO;

## Créditos

Esse tutorial foi criado após leitura e interpretação do tutorial fornecido pela própria Oracle em: [Docker Images on Github](https://github.com/oracle/docker-images) o qual possui melhor documentação e opções.
