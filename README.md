# Deploy do Kibana no Cluster EKS da AWS

Este guia fornece instruções abrangentes para realizar o deploy do Kibana em um cluster EKS (Amazon Elastic Kubernetes Service) na AWS, incluindo a configuração do Elasticsearch e a criação do cluster EKS com Terraform.

## Pré-requisitos

Antes de começar, certifique-se de ter o seguinte:

1. Uma conta na AWS com permissões adequadas para criar recursos como EKS, EC2, e VPC.
2. Terraform instalado em sua máquina local.
3. Uma nova máquina com Docker instalado.

## Implantar o EKS com Terraform

### Passo 1: Navegar até o Diretório do Terraform

Navegue até o diretório que contém os arquivos do Terraform:

```bash
cd terraform/terraforms/stg
```

### Passo 2: Inicializar o Projeto Terraform

Execute o comando abaixo para inicializar o projeto Terraform:s

```bash
terraform init
```

### Passo 3: Visualizar Alterações Propostas

Antes de aplicar as alterações, visualize o plano do Terraform:

```bash
terraform plan -out plan.out
```

### Passo 4: Aplicar Alterações

Se estiver satisfeito com as alterações propostas, aplique as alterações com o comando:

```bash
terraform apply plan.out
```

Após a conclusão, seu cluster EKS estará pronto para uso.

## Implantação do Kibana com Docker Compose

### Passo 1: Acessar a Nova Máquina

Acesse a nova máquina onde o Kibana será implantado.

### Passo 2: Instalar o Docker

Instale o Docker na nova máquina conforme as instruções do [site oficial do Docker](https://docs.docker.com/get-docker/).

### Passo 3: Criar o Docker Compose

Crie um arquivo chamado `docker-compose.yml` com o seguinte conteúdo:

```yaml
version: "3.8"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    container_name: elasticsearch
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - ${ES_PORT}:9200
    restart: always
    environment:
      - xpack.security.enabled=false
      - discovery.type=single-node
    ulimits:
      memlock:
        soft: -1
        hard: -1
  kibana:
    depends_on:
      - elasticsearch
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
    container_name: kibana
    volumes:
      - kibana-data:/usr/share/kibana/data
    ports:
      - ${KIBANA_PORT}:5601
    restart: always
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
volumes:
  elasticsearch-data:
    driver: local
  kibana-data:
    driver: local
```

### Passo 4: Criar o Arquivo `.env`

Crie um arquivo chamado `.env` no mesmo diretório do arquivo `docker-compose.yml` com o seguinte conteúdo:

```yaml
STACK_VERSION=8.11.1
ES_PORT=9200
KIBANA_PORT=5601
```

### Passo 5: Implantar Containers com Docker Compose

No diretório onde o arquivo docker-compose.yml está localizado, execute:

```bash
docker-compose up -d
```

Aguarde enquanto os containers do Elasticsearch e Kibana são baixados e iniciados.

### Passo 6: Verificar a Implantação

Verifique se os containers estão em execução:

```bash
docker ps
```

### Passo 7: Acessar o Kibana

Abra um navegador e acesse o endereço IP externo atribuído ao serviço Kibana, seguido da porta 5601. Você deve ver a interface do Kibana.

Agora você implantou o Kibana em seu cluster EKS usando Docker Compose na nova máquina. 🎉

**OBS**: Todos os scripts necessários para a implantação estão na pasta terraform no mesmo nível do README.
