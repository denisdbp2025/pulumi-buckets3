## Utilizando Pulumi para provisionar um Bucket no S3 ##

No meu caso estou usando o Windows junto ao Powershel.

Para instalar o Pulumi usei o Chocolatey.

Utilize o comando choco install pulumi

## Obs: Após a instalação para que o Pulumi funcione voce deve informar o Path onde se encontra o Pulumi,
basta digitar na pesquisa do menu iniciar do Windows Variáveis, e clicar em "Editar as variáveis de ambiente do sistema"
após isso na tela que se abrir clicar em "Variáveis de ambiente" e clica em Novo e informar o Path %USERPROFILE%\.pulumi\bin
com isso ja é possível usar o Pulumi.

Recomendo instalar o AWS CLI no Windows e depois de instalado executar o comando aws configure para informar sua Key e AccessKey
para confugurar sua conta, para obter essas chaves basta ir na seção de IAM na AWS e criar uma conta.

Crie uma pasta em qualquer local do seu computador e crie um arquivo Yaml de qualquer nome e coloque essas informações abaixo,
voce pode alterar o aws-teste por qualquer outro nome. Veja que aqui ele vai criar um recurso do S3 e subir um arquivo index.html
para seu bucket criado e ja vai retornar a URL para acesso ao bucket como se fosse um WebSite.

##############################################
name: aws-teste
description: aws-teste
runtime: yaml
resources:
  # Create an AWS resource (S3 Bucket)
  my-bucket:
    type: aws:s3:Bucket
  # Turn the bucket into a website:
  website:
    type: aws:s3:BucketWebsiteConfiguration
    properties:
      bucket: ${my-bucket.id}
      indexDocument:
        suffix: index.html
    # Create an S3 Bucket object
  index.html:
    type: aws:s3:BucketObject
    properties:
      bucket: ${my-bucket.id}
      source:
        fn::fileAsset: index.html
      contentType: text/html
      acl: public-read
    options:
      dependsOn:
        - ${ownership-controls}
        - ${public-access-block}

  # Permit access control configuration:
  ownership-controls:
    type: aws:s3:BucketOwnershipControls
    properties:
      bucket: ${my-bucket.id}
      rule:
        objectOwnership: ObjectWriter

  # Enable public access to the website:
  public-access-block:
    type: aws:s3:BucketPublicAccessBlock
    properties:
      bucket: ${my-bucket.id}
      blockPublicAcls: false
outputs:
  # Export the name of the bucket
  bucketName: ${my-bucket.id}
  url: http://${website.websiteEndpoint}
config:
  pulumi:tags:
    value:
      pulumi:template: aws-yaml
##############################################

Crie um arquivo no mesmo diretório chamado index.html e coloque as informações abaixo.
<html>
    <body>
        <h1>Hello, Pulumi!</h1>
    </body>
</html>

Feito isso podemos iniciar o Pulumi, escreva o comando pulumi new aws-yaml

Obs: aws-yaml pode ser trocado por outro nome.

O pulumi new comando guia você interativamente pelo processo de inicialização de um novo projeto, bem como pela criação e configuração de uma pilha (stack ). Uma pilha é uma instância do seu projeto, e você pode ter várias delas – como `project1`, `project2` , `project3` e ` project4` – cada uma com configurações diferentes.devstagingprod

Você será solicitado a inserir valores de configuração, como uma região da AWS. Você pode pressionar ENTER para aceitar o valor padrão us-east-1 ou digitar outro valor, como us-west-2:

The AWS region to deploy into (aws:region) (us-east-1): us-west-2
Se esta for a sua primeira vez executando o Pulumi, você será solicitado a fazer login no Pulumi Cloud. Este é um serviço gratuito, porém opcional, que facilita a Infraestrutura como Código (IaC) gerenciando o estado de forma segura e protegida.

Algumas informações:
Ao criar um recurso ele vai pedir o Yes para aprovar a solicitação.
Voce pode editar o arquivo Yaml e atualizar as informações usando pulumi up.
