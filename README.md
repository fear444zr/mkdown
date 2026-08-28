## Introducao

<br>

### O que e o EspoCRM?

O EspoCRM e uma webapp que permute que utilizadores, entrem e avaliem todas as conexoes da empresa de tipos diferentes e tambem funciona como um gestor de pessoas onde se podem atribuir tarefas e gerir o calendario apenas em uma plataforma.

<br>

### Documentacao

Este guia tem como objetivo auxiliar o operador a instalar o EspoCRM num servidor ubuntu. O EspoCRM e um CRM (Customer Relationship Manager) opensource 

<br>

## Corpo

<br>

### Downloads necessarios

Vamos precisar das seguintes packages:

```
apt install mysql-server apache2 unzip php
```

<br>

### Download do EspoCRM

Depois vamos aceder ao site do download do [EspoCRM](https://www.espocrm.com/)

Dentro do site vamos fazer download do ZIP do EspoCRM

<br>

### Enviar o ZIP para o servidor 

O ZIP pode ser enviado por SFTP (o SFTP vem default por SSH)

<br>

> Nota: Teremos que nos dirigir a diretoria onde esta o ficheiro pelo CMD/Terminal **ANTES** de fazer o SFTP
{.is-info}

<br>

```
cd </diretoria/do/ficheiro>
sftp user@servidor
put <ficheiro.zip>
```

<br>

### Rename do ficheiro, mover para a pasta do apache2 e extrair

<br>

> Nota: Teremos que mudar o "ficheiro.zip" pelo nome do ficheiro que fizemos upload para o servidor
{.is-info}

```
mv <ficheiro.zip> espocrm.zip
mv espocrm.zip /var/www/html'
cd /var/www/html
unzip espocrm.zip
```

Depois de extrair o ficheiro podemos remover o unzip

<br>

```
apt purge unzip
```
<br>

### Criar uma DB para o EspoCRM

<br>

```
mysql -uroot -p
```

<br>

> Nota: Teremos que mudar a "password" pela password correta
{.is-info}

<br>

```
create database espocrm character set utf8mb4 collate utf8mb4_bin;
create user espocrm@localhost identified by 'password';
grant all privileges on espocrm.* to espocrm@localhost;
quit;
```

<br>

### Configurar o Apache2 para o EspoCRM

Vamos ter que ativar o mod_rewrite para o apache2

```
sudo a2enmod rewrite
```

Depois vamos ao ficheiro de config do apache2

```
nano /etc/apache2/apache2.conf
```

E vamos adicionar a seguinte config:

```
DocumentRoot /var/www/html/espocrm/public
Alias /client/ /var/www/html/espocrm/client

<Directory /var/www/html/espocrm>
  AllowOverride All
</Directory>
```

Depois vamos dar restart ao apache2

```
sudo systemctl restart apache2
```

<br>

### Aceder ao Espocrm web install

<br>

> Nota: Teremos que trocar "host" pelo ip/hostname do servidor
{.is-info}


Agora que o espocrm ja esta corretamente instalado no servidor, podemos aceder ao web install em "https://host/espocrm"

<br>

## Troubleshooting

<br>

### Erro de Permissoes

<br>

> Caso exista o seguinte erro: "Permission denied for "data" ..." vamos inserir os comandos a baixo, caso nao apareca podemos continuar
{.is-danger}


```
cd /var/www/html/espocrm
sudo find data -type d -exec sudo chmod 775 {} + && sudo chown -R 33:33 .;
sudo systemctl restart apache2
```

<br>

## Conclusao

<br>

Apos seguir todos os passos deste guia o operador devera ter instalado com sucesso o EspoCRM no servidor e pode continuar a configuracao na seguinte pagina: [Configuracao do EspoCRM](http://192.168.1.10:3100/en/CEspoCRM)
