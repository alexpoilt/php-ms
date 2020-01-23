# Objetivos

O objetivo deste repositório é fornecer um exercício com o intuito de criar uma aplicação dependente de múltiplos contêineres. As informações relativas ao **Docker Compose**, **Kubernetes** e ao **Swarm** estão em seus respectivos diretórios.

Caso deseje, as respostas estão na branch **answers**.

# Microserviço

De maneira simplista, para que uma aplicação possa sobreviver dentro de um cluster de **Kubernetes** ou **Swarm** é preciso que ela seja escalável, mantendo seu comportamento ao incremento ou decremento de suas réplicas.

Isso implica algumas coisas simples de concepção, mas complexas de execução, como quebrar a aplicação em microserviços, persistir o estado fora da aplicação, fraca dependência e etc.

## Monolito

Monolito é uma aplicação que cuida de todas as tarefas do seu ecossitema, geralmente um sistema legado.

Agora imagine uma aplicação simples em PHP que valide acessos através de um formulário de login consultando um banco de dados MySQL e salve seus dados de sessão da maneira padrão, como arquivos no disco local.

Caso essa aplicação precise ser enviada para um outro **nó** do seu cluster o contêiner será destruído e recriado. Isso significa que todo o estado da aplicação foi perdido, e todos os usuários autenticados agora precisam autenticar-se novamente.

O mesmo ocorrerá caso algum problema aconteça com o contêiner da aplicação, ele será destruído e recriado, mesmo que seja na mesma máquina, um novo contêiner trás consigo uma nova camada.

O estado dessa aplicação deverá então ser colocado fora do contâiner, em um sistema de arquivo compartilhado, no banco de dados, ou a melhor das opções, um servidor de **cache**.

### Aplicação - PHP

Esta aplicação não é um microserviço, mas serve para entender os conceitos que envolvem a criação de aplicações mais complexas dentro de orquestradores como **Kubernetes** ou **Swarm**.
O código utiliza 5 variáveis de ambiente para se conectar ao banco de dados:

 - DB_HOST
 - DB_PORT
 - DB_USER
 - DB_NAME
 - DB_PASS

O PHP utiliza o *php.ini* para configurar o servidor externo de cache que neste caso é o **memcached**. Para que esta conexão funcione, é necessário adicionar no *php.ini*:

```ini
[Session]
session.save_handler = memcached
session.save_path = "m1:11211,m2:11211,m3:11211...m9:11211"
```

Uma imagem de contêiner deve ser construída para a aplicação. Para facilitar, as dependências, um diretório de exemplo e o **CMD** estão descritos abaixo:

```docker
RUN apk add --no-cache php-cli php-mysqli php-session php7-pecl-memcached && mkdir /app
CMD ["php", "-S", "0.0.0.0:8080", "-t", "/app"]
```

**Obs:** Existe um arquivo de exemplo com o mínimo necessário de configurações do `php.ini` no diretório **files**.

### Banco de Dados - MySQL

Pela facilidade de configuração recomendo que se utilize o **MySQL** versão 5.7. Versões mais novas ou antigas podem ser utilizadas, desde que se faça os ajustes necessários. Será preciso popular o banco com os dados presentes em `files/dump.sql`. Neste arquivo há algumas dicas de como essa tarefa pode ser realizada.

**Obs:** O ideal é que os dados do banco sejam persistidos em algum volume.

### Sessão - Memcached

A sessão será gerenciada pelo **Memcached** sem maiores configurações desde que a aplicação consiga acessá-lo de alguma forma através da rede.

### Usuários

Os usuários para testar o login estão no arquivo `files/dump.sql`. Seus e-mails e senha são:

- paramahansa@yogananda.in - 123
- victor@frankenstein.co.uk - 123
