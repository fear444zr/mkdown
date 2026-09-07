## Introducao

<br>

Este guia tem como objetivo auxiliar o operador a instalar o TheHive num servidor ubuntu.

<br>

## Corpo

<br>

### Instalar o Docker
<br>
A instalacao do TheHive4 (ultima versao opensource) apenas existe em docker, por isso antes de fazer o container com o docker compose vamos ter que instalar o docker com a documentacao deles:

[Docker install Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

<br>

### Criar as diretorias e os ficheiros necessarios para o cortex-jobs

<br>

Criacao das pastas e dos ficheiros:

```
sudo mkdir /opt/cortex-jobs
```

```
mkdir cortex-elastic
cd cortex-elastic
mkdir cortex
mkdir cortex/logs
mkdir application.conf
touch docker-compose.yml
```

<br>

Vamos abrir o docker-compose.yml:

```
nano docker-compose.yml
```
<br>

Vamos dar paste a seguinte config:
```
volumes:
  cortex_data:
  es_data:

services:
  elasticsearch:
    image: 'elasticsearch:7.11.1'
    container_name: elasticserch
    environment:
      - http.host=0.0.0.0
      - discovery.type=single-node
      - script.allowed_types=inline
      - thread_pool.search.queue_size=100000
      - thread_pool.write.queue_size=10000
    ports:
      - "0.0.0.0:9200:9200"

  cortex:
    image: thehiveproject/cortex:latest
    container_name: cortex
    restart: unless-stopped
    environment:
      - job_directory=/tmp/cortex-jobs
      - docker_job_directory=/opt/cortex-jobs
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /opt/cortex-jobs:/tmp/cortex-jobs
      - ./cortex/logs:/var/log/cortex
      - ./cortex/application.conf:/cortex/application.conf
    depends_on:
      - elasticsearch
    ports:
      - "0.0.0.0:9001:9001"
```

### Depois vamos criar as diretorias e os ficheiros para o TheHive4

<br>

Criacao das pastas e dos ficheiros:

```
mkdir thehive4
cd thehive4
mkdir vol
mkdir vol/thehive
mkdir vol/thehive/data
mkdir vol/thehive/index
touch vol/thehive/application.conf
touch docker-compose.yml
```

Vamos abrir o application.conf:

```
nano vol/thehive/application.conf
```

Vamos dar paste a seguinte config:

```
# Secret Key
# The secret key is used to secure cryptographic functions.
# WARNING: If you deploy your application on several servers, make sure to use the same key.
#play.http.secret.key="xNq1iBhCelVfDjKqJgZYUBv75cSqJ1tqkgu33jnIkrwmo3LIremIwfFNOlcoSv78"

# Elasticsearch
search {
  ## Basic configuration
  # Index name.
   index = thehive
  # ElasticSearch cluster name.
    cluster = hive
  # ElasticSearch instance address.
    host = ["10.0.0.22"]

  # Enable SSL to connect to ElasticSearch
    search.ssl.enabled = false
}

#################################################################
####################################################################


# Authentication
#auth {
	# "provider" parameter contains authentication provider. It can be multi-valued (useful for migration)
	# available auth types are:
	# services.LocalAuthSrv : passwords are stored in user entity (in Elasticsearch). No configuration is required.

#	provider = [local]

  # By default, basic authentication is disabled. You can enable it by setting "method.basic" to true.
  #method.basic = true

#}

# Maximum time between two requests without requesting authentication
#session {
#  warning = 55m
#  inactivity = 1h
#}


# Max textual content length
play.http.parser.maxMemoryBuffer= 1M
# Max file size
play.http.parser.maxDiskBuffer = 1G


#######################################################################################

## CORTEX configuration
# More information at https://github.com/TheHive-Project/TheHiveDocs/TheHive4/Administration/Connectors.md
# Enable Cortex connector
play.modules.enabled += org.thp.thehive.connector.cortex.CortexModule
cortex {
  servers: [
    {
      name: "Cortex"                # Cortex name
      url: "http://10.0.0.22:9001" # URL of Cortex instance
      auth {
        type: "bearer"
        key: "nwK1Pvzw8K8H5htjhGe//p/GYH4aW9yK"                 # Cortex API key
      }
      #wsConfig { ssl.loose.acceptAnyCertificate = true }                  # HTTP client configuration (SSL and proxy)
    }
  ]
}



## MISP configuration
# More information at https://github.com/TheHive-Project/TheHiveDocs/TheHive4/Administration/Connectors.md
# Enable MISP connector
play.modules.enabled += org.thp.thehive.connector.misp.MispModule
misp {
 interval: 1 hour
 servers: [
   {
     name = "misp_server"            # MISP name
     url = "https://10.0.0.21" # URL or MISP
     auth {
       type = key
       key = "4Ns01HbJSdje6ZqsXc5L6F4SWDGakhRZhzYEfrSb"             # MISP API key
     }
     wsConfig { ssl.loose.acceptAnyCertificate = true   }               # HTTP client configuration (SSL and proxy)
   }
 ]
}
```

Vamos abrir o docker-compose.yml:

```
nano docker-compose.yml
```

E vamos dar paste da seguinte config:

```
services:
  thehive:
    image: thehiveproject/thehive4:latest
    container_name: thehive
    restart: unless-stopped
    depends_on:
      - cassandra
    mem_limit: 2000m
    ports:
      - "0.0.0.0:9000:9000"
    environment:
      - JVM_OPTS="-Xms2048M -Xmx2048M"
      - MAX_HEAP_SIZE=1G
      - HEAP_NEWSIZE=1G
    volumes:
      - ./vol/thehive/application.conf:/etc/thehive/application.conf
      - ./vol/thehive/data:/opt/thp/thehive/data
      - ./vol/thehive/index:/opt/thp/thehive/index
    networks:
      - soc_network

  cassandra:
    image: 'cassandra:4'
    container_name: cassandra
    restart: unless-stopped
    ports:
      - "0.0.0.0:9042:9042"
    environment:
      - CASSANDRA_CLUSTER_NAME=TheHive
    volumes:
      - cassandradata:/var/lib/cassandra
    networks:
      - soc_network

  redis:
    image: redis:latest
    networks:
      - soc_network

volumes:
  miniodata:
  cassandradata:
  elasticsearchdata:
  thehivedata:


networks:
    soc_network:
          driver: bridge
```

<br>

### Alterar o ficheiro de config

Antes de fazer o deploy queremos configurar o application.conf do thehive4

<br>

```
nano application.conf
```
<br>

### Dar compose aos containers

<br>

> Pode demorar alguns minutos
{.is-info}

<br>

Antes de dar compose vamos voltar para a pasta do cortex:

```
cd ..
cd cortex-elastic
```

Agora sim vamos dar compose:

```
docker compose up
```

<br>

Quando o primeiro compose tiver acabado, vamos trocar de pasta para a pasta do thehive

```
cd ..
cd thehive4
```

E vamos novamente dar compose:

```
docker compose up
```

Agora podemos confirmar que os containers foram instalados:

```
docker ps
```

No output deste comando devemos conseguir ver todos os containers e devem de estar todos up

<br>

### Aceder ao TheHive

<br>

> Trocar "host" pelo hostname/ip da maquina
{.is-info}


Apos podemos aceder ao TheHive na porta 9000 <http://host:9000>

## Conclusao

<br>

Apos seguir todos os passos deste guia o operador devera ter instalado com sucesso o TheHive com o docker, pode continuar a configuracao na seguinte pagina: [Configuracao do TheHive](http://192.168.1.10:3100/en/CTheHive)
