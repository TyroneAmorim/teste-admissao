### Olá, prezados! :wave:

\
Estou muito satisfeito por ter cumprido o desafio no prazo estimado e conseguir alcançar todos os requisitos apresentados.
A aplicação está dividida em FRONT e BACK, em dois repositórios diferentes. Foi desenhada da seguinte maneira: 

- Página de Login: Onde os usuários já cadastrados conseguem acesso ao sistema.
- Página de cadastro: Onde os novos usuários podem fazer seu cadastro.
- Página inicial: É uma parte mais visual, onde há uma breve descrição do sistema e seu propósito.
- Página de nova operação: Onde é possível que um usuário faça o lançamento de uma operação, informando os dados necessários.
- Página de operações: Onde é possível acompanhar todas as operações realizadas pelo próprio usuário.
- Página de configurações: Onde o usuário pode atualizar suas informações pessoais, ou de acesso.
- Página de pacotes: Onde é possível acompanhar todos os pacotes que estão relacionados com aquele usuário e suas informações.

## Idéia para expansão
Com estes endpoints já criados, é possível fazer um sistema de notificação em tempo real ou alimentar outro sistema, para o gerenciamento da separação e  transporte dos pacotes.


## Foco do desenvolvimento

Procurei seguir como premissa o padrão e coesão de código, e escrever o mínimo possível. Por conta disso também, fiz a escolha de ferramentas que pode ser acompanhada abaixo.

## Ferramentas e tecnologias

- :gear: [React](https://pt-br.reactjs.org/)
  - :hammer_and_wrench: [axios](https://axios-http.com/ptbr/)
  - :hammer_and_wrench: [antd](https://ant.design/)
- :gear: [Node](https://nodejs.org/en/)
  - :hammer_and_wrench: [typeorm](https://typeorm.io/)
  - :hammer_and_wrench: [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken)
  - :hammer_and_wrench: [dotenv](https://www.npmjs.com/package/dotenv)
  - :hammer_and_wrench: [class-validator](https://github.com/typestack/class-validator)
  - :hammer_and_wrench: [class-transformer](https://github.com/typestack/class-transformer)
  - :hammer_and_wrench: [nestjs](https://nestjs.com/)
- :hammer_and_wrench: [eslint](https://eslint.org/)
- :hammer_and_wrench: [prettier](https://prettier.io/)
- :gear: [typescript](https://www.typescriptlang.org/)
- :gear: [postgresql](https://www.postgresql.org/)
- :gear: [docker](https://www.docker.com/)
- :hammer_and_wrench: [vscode](https://code.visualstudio.com/)

## Instalação e execução da aplicação
- Repositório [Front](https://github.com/TyroneAmorim/frente-co-front)
- Repositório [Back](https://github.com/TyroneAmorim/frente-co-back)

1. Clone o Repositório do [Back](https://github.com/TyroneAmorim/frente-co-back)
```console
git clone https://github.com/TyroneAmorim/frente-co-back.git
```
2. Entre no diretório do Back
```console
cd $PATH/frente-co-back
```
3. Rode o comando para gerar a imagem do Back no Docker
```console
docker build --no-cache -t node-api .
```
4. Clone o Repositório do [Front](https://github.com/TyroneAmorim/frente-co-front)
```console
git clone https://github.com/TyroneAmorim/frente-co-front.git
```
5. Entre no diretório do Front
```console
cd $PATH/frente-co-front
```
6. Rode o comando para gerar a imagem do Front no Docker
```console
docker build -t site-react .
```
7. Entre na pasta dos arquivos de tabela do banco de dados
```console
cd $PATH/frente-co-back/sql
```
8. Crie o arquivo Dockerfile com o seguinte conteúdo
```dockerfile
FROM debian

RUN apt-get update

RUN apt-get -y install postgresql

COPY ["*sql", "/tmp/"]

USER postgres

# Create user and database
RUN    /etc/init.d/postgresql start &&\
    psql --command "CREATE USER frentecorret WITH SUPERUSER PASSWORD 'frentecorret';" &&\
    createdb -O postgres frentecorret &&\
    psql --command "CREATE SCHEMA frentecorret AUTHORIZATION postgres;" &&\
    psql -d frentecorret &&\
    psql -f /tmp/address__DDL.sql &&\
    psql -f /tmp/package__DDL.sql &&\
    psql -f /tmp/client__DDL.sql  &&\
    psql -f /tmp/operation__DDL.sql

# Adjust PostgreSQL configuration so that remote connections to the
# database are possible.
RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/13/main/pg_hba.conf

# And add ``listen_addresses`` to ``/etc/postgresql/13/main/postgresql.conf``
RUN echo "listen_addresses='*'" >> /etc/postgresql/13/main/postgresql.conf

# Expose the PostgreSQL port
EXPOSE 5432

# Add VOLUMEs to allow backup of config, logs and databases
VOLUME  ["/etc/postgresql", "/var/log/postgresql", "/var/lib/postgresql"]

# Set the default command to run when starting the container
CMD ["/usr/lib/postgresql/13/bin/postgres", "-D", "/var/lib/postgresql/13/main", "-c", "config_file=/etc/postgresql/13/main/postgresql.conf"]
```
9. Rode o comando para criar a instância do banco de dados
```console
docker build --no-cache -t db-api .
```
10. Crie o arquivo 'docker-compose.yml' com o conteúdo
```yml
version: "3.3"

services:
    db:
        container_name: db-api
        image: db-api
        ports:
            - "5432:5432"
        volumes: 
            - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
            - "dbdata:/var/lib/postgresql/data"

    api:
        container_name: node-api
        image: node-api
        ports:
            - "4000:4000"
        depends_on: 
            - db
        command:  npx ts-node src/main.ts
        environment: 
            - NODE_ENV=production
            - DB_TYPE=postgres
            - DB_HOST=db-api
            - DB_PORT=5432
            - DB_USER=frentecorret
            - DB_PASSWORD=frentecorret
            - DB_NAME=postgres
            - DB_SCHEMA=frentecorret
            - JWT_SECRET=0BD29231D7946251B4355371635415A00E08CFB0
            - LIMIT_MONEY_PAPER_PACKAGE=50
            - API_PORT=4000

    web:
        container_name: site-react
        image: site-react
        ports:
            - "8080:80"
        depends_on: 
            - api 
            - db
        environment:
            - REACT_APP_API_URL=http://node-api:4000/
            - REACT_APP_LIMIT_OPERATION=5000
            - REACT_APP_PACKAGE_LIMIT_MONEY=50
volumes:
    dbdata:
```
11. Rode o comando para gerar os contâiners
```console
docker-compose up --force-recreate
```
12. E por fim acesse a aplicação em: [http://localhost:8080](http://localhost:8080)

# Aviso!!
As variáveis de ambiente com informações sensíveis só estão presentes aqui, por conta de o projeto se trara de um teste, gerando uma praticidade na hora de configurar o ambiente.
