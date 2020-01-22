# Swarm

Crie um cluster de **swarm** com três máquinas. É possível utilizar o **docker-machine** ou mesmo o **vagrant**.

A aplicação deve ser provisionada com um **compose-file** em um único *stack*.

## Expor as aplicaão

Não exponha nenhuma outra aplicação além do contêiner com php. Expor o **banco de dados** ou o **memcached** é altamente arriscado.

## Acesso entre as aplicações

Lembre-se que o **swarm** fornece um DNS interno capaz de resolver as aplicações pelo próprio nome.

## Compartilhar arquivo - php.ini

Como o contêiner do PHP utiliza-se do `php.ini` para encontrar os outros contêineres do **memcached**, torna-se uma boa prática criar um **config** para o `php.ini` e montá-lo em todos os contêineres da aplicação.
Utilizando um **config** é possível fazer alterações apenas neste "arquivo compartilhado", por exemplo adicionando novos contêineres do **memcached**, sem precisar criar novas imagens

## Dados de acesso ao Banco

Como os dados de acesso ao banco são sigilosos e podem ser compartilhados entre mais de uma aplicação, é uma boa prática definir esses dados de acesso dentro de um **secret** e então utilizá-lo para popular as variáveis de ambiente de cada contêiner, ou montá-lo como arquivo.
Com o secret criado, tanto o banco de dados como a a aplicação podem usufruir dos mesmos dados, facilitando a manutenção.

## Volumes

Para os volumes compartilhados a forma mais simples é criar uma máquina exportando diretórios via **NFS**. Suponhando que a máquina tenha o endereço **172.27.11.40** o trecho do **compose-file** poderia ser definido dessa forma:

```yaml
volumes:
  nfs_mysql_data:
    driver_opts:
      type: nfs
      o: addr=172.27.11.40,nolock,soft,rw
      device: ':/srv/nfs/mysql_data'
```

**Obs:** As opções do volume definidas na chave `o:` são obrigatórias, do contrário o contêiner não consegueirá montar o volume.

## Tarefas

1. Provisionar um cluster de swarm.
2. Criar um **Dockerfile**, criar e enviar a imagem para o Docker Hub.
3. Criar um **config** para o `php.ini`.
4. Criar um **secret** para os dados do banco.
5. Criar uma quarta máquina e criar um **compartilhamento via NFS**.
6. Criar um **compose-file** com:
  - MySQL - utilizar volume via NFS para persistir dados
  - Memcached1 - não utilizar réplica
  - Memcached2 - não utilizar réplica
  - Aplicação - 3 réplicas, publicar na porta 8080.
7. Provisionar a aplicação no cluster e testar o login.
8. Executar o *teste de alta disponibilidade*

## Teste de alta disponibilidade

Este script executa requisições em loop no mesmo endereço, e não deve apresentar falhas pois a aplicação está configurada - de forma simplista - para alta disponibilidade:

**ha-test.yml**

```bash
#!/bin/bash

# O teste está incompleto
while true; do
  curl localhost:8080;
  sleep 1;
done
```

### Dicas

Tanto o **config** como o **secret**, assim como o **volume**, podem ser criados em um único **compose-file**.
