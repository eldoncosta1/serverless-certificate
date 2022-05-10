# Serverless - AWS Node.js Typescript

```jsx
serverless create --template aws-nodejs-typescript --path certificate
```

## Plugins para trabalhar com serverless offline

```jsx
yarn add serverless-offline -D
```

## Configurando serverless.yml

```jsx
service:
  name: certificate

plugins:
  - serverless-offline
  - serverless-webpack

provider:
  name: aws
  region: sa-east-1
  runtime: nodejs14.x

custom:
  webpack:
    webpackConfig: ./webpack.config.js
    includeModules: true

functions:
  hello:
    handler: src/functions/hello.handle
```

## Configurando AWS para serviços

```jsx
devfullstack10@gmail.com

Usuario AWS
serverlessnodejs

ID CHAVE DE ACESSO

gerar id de chave de acesso na aws
```

## Configurando aws no serverless

```jsx
serverless config credentials --provider aws --key='key gerada no usuario aws' --secret 'secret gerada no usuario aws'

Caso já exista credenciais configuradas utilizar a flag -o ao final do comando acima
```

## Enviando função para aws

```jsx
Criação de script no package.json para deploy

"deploy": "serverless deploy --verbose"
```

## Configurando o DynamoDB

```jsx
yarn add serverless-dynamodb-local -D

Configurando no serverless.yml

resourcers:
  Resources:
    dbCertificateUsers:
      Type: AWS::DynamoDB::Table
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5
      Properties:
        TableName: users_certificates
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH

Instalando dynamodb na aplicação

serverless dynamodb install

Startando banco de dados

serverless dynamodb start
```

## Template para transformar inserir variáveis dinâmicas em documento

```jsx
yarn add handlebars

yarn add chrome-aws-lambda (permite consegue visualizar pdf no browser)
yarn add puppeteer-core

obs: a versão que funcionou foi
"chrome-aws-lambda": "^6.0.0",
"puppeteer-core": "^6.0.0",

yarn add dayjs

yarn add copy-webpack-plugin -D
```

## Serverless.yml completo

```jsx
service: certificate

plugins:
  - serverless-webpack
  - serverless-dynamodb-local
  - serverless-offline

custom:
  webpack:
    webpackConfig: "webpack.config.js"
    includeModules: true
  dynamodb:
    stages:
      - dev
      - local
    start:
      port: 8000
      inMemory: true
      migrate: true
  bucket: certificatecourses

provider:
  name: aws
  region: sa-east-1
  runtime: nodejs14.x
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:*
      Resource: "*"
    - Effect: Allow
      Action:
        - s3:*
      Resource: "*"

functions:
  generateCertificate:
    handler: src/functions/generateCertificate.handle
    events:
      - http:
          path: generateCertificate
          method: POST
    iamRoleStatements:
      - Effect: Allow
        Action:
          - dynamodb:Query
          - dynamodb:PutItem
        Resource: "arn:aws:dynamodb::${self.provider.region}:*:table/users_certificates"
  verifyCertificate:
    handler: src/functions/verifyCertificate.handle
    events:
      - http:
          path: verifyCertificate/{id}
          method: GET
          cors: true
    iamRoleStatements:
      - Effect: Allow
        Action:
          - dynamodb:Query
        Resource: "arn:aws:dynamodb::${self.provider.region}:*:table/users_certificates"
resources:
  Resources:
    dbCertificateUsers:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: users_certificates
        ProvisionedThroughput:
          ReadCapacityUnits: 5
          WriteCapacityUnits: 5
        AttributeDefinitions:
          - AttributeName: id
            AttributeType: S
        KeySchema:
          - AttributeName: id
            KeyType: HASH
```
