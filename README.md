## Introducao

<br>

Este guia tem como objetivo auxiliar o operador a instalar o zabbix num servidor ubuntu.

<br>

## Corpo

<br>

### Trocar para root user
<br>

Antes de fazer alguma alteracao queremos passar para root user:

```
sudo -s 
```
<br>

### Instalar a diretoria do Zabbix

<br>

Vamos aceder ao site do download do [Zabbix](https://www.zabbix.com/download?)

Dentro deste site vamos selecionar os parametros necessarios conforme o servidor onde vamos instalar e vamos copiar o link de download. Esta documentacao tem foco na versao de ubuntu do zabbix.

<br>

> Exemplo: "wget https://repo.zabbix.com/zabbix/7.4/release/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.4+ubuntu26.04_all.deb"

<br>

Depois de fazer o download vamos adicionar o repositorio do Zabbix e atualizar

```
dpkg -i <zabbix_package>
apt update
```

<br>

### Instalar o Zabbix server, agent e frontend

<br>

```
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

<br>

### Criar a DB

<br>

```
mysql -uroot -p
```
<br>

> Os proximos comandos serao feitos dentro do MySQL
{.is-info}

<br>

```
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

<br>

###  Importar dados e esquema inicial do Zabbix

<br>

> Este comando pode demorar alguns minutos:
{.is-info}


```
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix 
```

<br>

### Desativar o log_bin_trust_function_creators

<br>

```
mysql -uroot -p
set global log_bin_trust_function_creators = 0;
quit; 
```

<br>

### Adicionar password a DB do Zabbix

<br>

```
DBPassword=password
```

<br>

### Comecar os servicos necessarios e adicina-los ao startup

<br>

```
systemctl restart zabbix-server zabbix-agent apache2 php8.5-fpm
systemctl enable zabbix-server zabbix-agent apache2 php8.5-fpm 
```

### Abrir o Zabbix no web browser

<br>

O URL default do Zabbix apos a configuracao inicial e o http://host/zabbix 
Apos o web setup o user e a password default sao respetivamente "Admin" e "zabbix"

## Troubleshoot

<br>

### Erro MySQL

<br>

> ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)
{.is-danger}

Para corrigir este erro apenas necessitamos de instalar o package do mysql-server:


```
apt install mysql-server
```
<br>

### Erro Apache2

<br>

> Job for apache2.service failed because the control process exited with error code.
{.is-danger}

<br>

Para corrigir este erro precisamos de perceber qual e a causa do apache nao ligar:

```
systemctl status apache2
```

No meu caso foi possivel verificar que ja existia outra aplicacao a correr na porta 80:

> apachectl[17070]: (98)Address already in use: AH00072: make_sock: could not bind to address [::]:80
{.is-danger}

<br>

De seguida tentei perceber o que estava a correr na porta 80:

```
sudo lsof -i : 80
```

<br>

Verifiquei que era o caddy que estava a causar conflito com o apache na porta 80, entao removi o caddy:

```
apt purge caddy
```

<br>

## Conclusao

<br>

Apos seguir todos os passos deste guia o operador devera ter instalado com sucesso o zabbix num servidor ubuntu, pode continuar a configuracao na seguinte pagina: [Configuracao do Zabbix](http://192.168.1.10:3100/en/CZabbix)
