# Docker Compose

Crie uma máquina virtual e instale a versão mais recente do Docker. É possível utilizar o **docker-machine** ou mesmo o **vagrant**.

A aplicação deve ser provisionada com um único **compose-file**.

## Expor as aplicaão

Não exponha nenhuma outra aplicação além do contêiner com php. Expor o **banco de dados** ou o **memcached** é altamente arriscado.

## Acesso entre as aplicações

Lembre-se que o **docker compose** fornece um DNS interno capaz de resolver as aplicações pelo próprio nome.

## Modificar o arquivo - php.ini

Como o contêiner do PHP utiliza-se do `php.ini` para encontrar os outros contêineres do **memcached**, para que o a aplicação possa fazer isso, será preciso adicionar um arquivo `php.ini` modificado durante a criação da imagem.

O arquivo `php.ini` deve ser montado dentro do contêiner da aplicação em `/etc/php7/php.ini`.

## Volumes

Persista os dados do MySQL em um volume gerenciado pelo Docker. É necessário que os dados resistam a constante criação e destruição dos contêineres.

## Tarefas

1. Provisionar uma máquina virtual com Docker.
2. Criar um **Dockerfile**, criar e enviar a imagem para o Docker Hub.
3. Criar um **compose-file** com:
  - MySQL - utilizar volume via NFS para persistir dados
  - Memcached1 - não utilizar réplica
  - Memcached2 - não utilizar réplica
  - Aplicação - 3 réplicas, publicar na porta 8080.
4. Provisionar a aplicação na máquina e testar o login.
5. Executar o *teste de alta disponibilidade*

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
