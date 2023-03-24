# DIO DynamoDb - Desafio de Projeto

Repositório para o desafio de projeto utilizando o DynamoDB

### Serviços utilizado
  - Amazon DynamoDB
  - Amazon CLI para execução em linha de comando
  - Docker
  

### Comandos para execução do experimento:


- Criar container com dynamoDb localamente:

No diretório do arquivo "docker-compose.yml" executar o comando para que o container de dynamodb seja criado:
```sh
docker-compose up --build
```

- Criar uma tabela

```
aws dynamodb create-table \
    --table-name Carro \
    --attribute-definition \
      AttributeName=Modelo,AttributeType=S \
      AttributeName=Ano,AttributeType=S \
    --key-schema \
      AttributeName=Modelo,KeyType=HASH \
      AttributeName=Ano,KeyType=RANGE \
   --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
   --endpoint-url http://localhost:8000
```

- Inserir um item

```
aws dynamodb put-item \
    --table-name Carro \
    --item file://itemCarro.json \
    --endpoint-url http://localhost:8000
```

- Inserir múltiplos itens

```
aws dynamodb batch-write-item \
    --request-items file://batchcarro.json \
    --endpoint-url http://localhost:8000
```

- Criar um index global secundário baeado na cor do carro

```
aws dynamodb update-table \
    --table-name Carro \
    --attribute-definitions AttributeName=Cor,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Cor-index\",\"KeySchema\":[{\"AttributeName\":\"Cor\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --endpoint-url http://localhost:8000
```

- Criar um index global secundário baseado no modelo e cor do carro

```
aws dynamodb update-table \
    --table-name Carro \
    --attribute-definitions\
        AttributeName=Model,AttributeType=S \
        AttributeName=Cor,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"ModeloCor-index\",\"KeySchema\":[{\"AttributeName\":\"Modelo\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Cor\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --endpoint-url http://localhost:8000

```

- Criar um index global secundário baseado no ano e na marca do carro

```
aws dynamodb update-table \
    --table-name Carro \
    --attribute-definitions\
        AttributeName=Ano,AttributeType=S \
        AttributeName=Marca,AttributeType=S \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"AnoMarca-index\",\"KeySchema\":[{\"AttributeName\":\"Ano\",\"KeyType\":\"HASH\"}, {\"AttributeName\":\"Marca\",\"KeyType\":\"RANGE\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]" \
    --endpoint-url http://localhost:8000
```

- Pesquisar item por modelo

```
aws dynamodb query \
    --table-name Carro \
    --key-condition-expression "Modelo = :modelo" \
    --expression-attribute-values  '{":modelo":{"S":"Gol"}}' \
    --endpoint-url http://localhost:8000
```
- Pesquisar item por modelo e ano

```
aws dynamodb query \
    --table-name Carro \
    --key-condition-expression "Modelo = :modelo and Ano = :ano" \
    --expression-attribute-values file://keyconditions.json \
    --endpoint-url http://localhost:8000
```

- Pesquisa pelo index secundário baseado na cor

```
aws dynamodb query \
    --table-name Carro \
    --index-name Cor-index \
    --key-condition-expression "Cor = :name" \
    --expression-attribute-values  '{":name":{"S":"Azul"}}' \
    --endpoint-url http://localhost:8000
```

- Pesquisa pelo index secundário baseado no modelo e cor do carro

```
aws dynamodb query \
    --table-name Carro \
    --index-name ModeloCor-index \
    --key-condition-expression "Modelo = :v_modelo and Cor = :v_cor" \
    --expression-attribute-values  '{":v_modelo":{"S":"HB20"},":v_cor":{"S":"Azul"} }' \
    --endpoint-url http://localhost:8000
```

- Pesquisa pelo index secundário baseado no ano e marca do carro

```
aws dynamodb query \
    --table-name Carro \
    --index-name AnoMarca-index \
    --key-condition-expression "Ano = :v_ano and Marca = :v_marca" \
    --expression-attribute-values  '{":v_ano":{"S":"2013"},":v_marca":{"S":"Volkswagen"} }' \
    --endpoint-url http://localhost:8000
```

## Observações
No arquivo "docker-compose.yml" foi definido que o container seria liberado porta 8000, por conta disso é importante sempre utilizar o comando ao final dos comando do AWS CLI
```
--endpoint-url http://localhost:8000
```

## Referências
Projeto baseado no fork do [projeto](https://github.com/cassianobrexbit/dio-live-dynamodb)
