<h1>AWS Lambda para EC2 com Python</h1>

![DIAGRAMA1](https://github.com/user-attachments/assets/5ded9eea-f7bc-4b2b-a8ea-3202d67eb8f9)

<h2>Objetivo</h2>

Utilizar o serviço AWS Lambda para configurar um provisionamento de máquinas EC2 através de um script Python. Demonstrar a eficiência e velocidade para criar processos de automação, integrando diversos serviços da plataforma em Nuvem. 

<h3>📜Passo a passo</h3>

<h4> 1. Primeiro devemos criar o Key Pair para permitir o acesso à instância. Para isso devemos navegar até o console EC2 e encontrar a opção no Menu lateral. </h4>

![EC2MENU1](https://github.com/user-attachments/assets/a17c216c-a640-4fd3-8c98-b77071cc5da5)
![keypairsmenu2](https://github.com/user-attachments/assets/3a7ebbdc-c939-4181-959a-1e418f6f3225)

![createkeybotao3](https://github.com/user-attachments/assets/54a61111-c487-446c-8844-3557d4ad9b4d)

Selecionamos a opção "Create key pair" e progredimos para a tela de configuração, onde iremos associar um nome para nossa chave, bem como escolher a extensão para acesso (no meu caso gosto de usar o Putty).

![telakey4](https://github.com/user-attachments/assets/73a0da66-7811-45ef-b1ce-959b9e066644)

Após salvar a chave em um diretório seguro do seu computador, iremos para o serviço de Lambda. 

<h4> 2. AWS Lambda </h4>

![lambdamenu6](https://github.com/user-attachments/assets/394af0ac-427f-4ee2-a540-1557b0c185a8)

Selecione o botão "Create function" para entrar na tela de configuração 

![image](https://github.com/user-attachments/assets/c388cd42-aa93-49cc-9de1-84d4705486f3)

Devemos selecionar a versão do Python que iremos trabalhar, a arquitetura e também marcar a opção para que seja criada uma "Role" básica no IAM, garantindo as permissões iniciais para o serviço. 

![lambdacriando7](https://github.com/user-attachments/assets/6502b0dd-bd31-4aab-a6e0-7242d4ef8419)

Navegando para baixo na página encontramos a seção de configuração, onde se encontra a "Role" que criamos.

![permissoes8](https://github.com/user-attachments/assets/a41d59cc-ab9a-4a24-8105-7edc204c088d)

Ao clicar sobre a Role, nos redirecionaremos para o console IAM, onde vamos encontrar a seção de edição de politica via arquivo JSON. 

![image](https://github.com/user-attachments/assets/3c732dd0-334c-4eb0-a1f8-fc50aa88525b)

Ao abrir o editor, podemos configurar as permissões que precisamos nessa "Role". No caso será adiciona a permissão de <i>Logs</i> para auditarmos qualquer erro que possa ocorrer na execução, e também a permissão <i>ec2:RunInstances</i> para a criação das Instâncias. 

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Action": [
        "ec2:RunInstances"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}


```

Podemos revisar as mudanças e verificar que já foi identificado as solicitações que fizemos. 

![reviewpol10](https://github.com/user-attachments/assets/f44a330b-2a07-40ae-8061-2bae97873429)

<h4> 3. Criação do Script Python </h4>

Vamos voltar até o editor de código onde iremos colocar o Script.

![image](https://github.com/user-attachments/assets/5b288bf1-f8f2-407e-a0d7-0cd7f5ea024d)

Começamos importando o boto3, o SDK da AWS para trabalhar com Python. Vamos criar quatro variáveis de ambiente no ambiente da Lambda.

- AMI: ID da Amazon Machine Image (AMI) que será usada para criar a instância EC2.
- INSTANCE_TYPE: Tipo da instância EC2 (por exemplo, t2.micro).
- KEY_NAME: Nome do par de chaves (KeyPair) que será usado para acessar a instância via SSH.
- SUBNET_ID: ID da sub-rede onde a instância será criada (parte da VPC).

```

import os 
import boto3

AMI = os.environ['AMI']
INSTANCE_TYPE = os.environ['INSTANCE_TYPE']
KEY_NAME = os.environ['KEY_NAME']
SUBNET_ID = os.environ['SUBNET_ID']

ec2 = boto3.resource('ec2')

def lambda_handler(event, context):
    
    instance = ec2.create_instances( 
        ImageId=AMI, 
        InstanceType=INSTANCE_TYPE,
        KeyName=KEY_NAME,
        SubnetId=SUBNET_ID,
        MaxCount=1, 
        MinCount=1
    )

    print("New instance created:", instance[0].id)

```
Após acessarmos o recurso EC2 pela biblioteca do boto3, declaramos nossa função principal. Nela é feita a criação da instância passando os parâmetros que coletamos anteriormente através das variáveis.
Vamos acessar a configuração de variáveis de ambiente, fornecendo os dados necessários para a criação. 

![envvariables12](https://github.com/user-attachments/assets/053beeba-cd45-4c0e-9492-1ce6f9f1dda1)
![passandovariaveis13](https://github.com/user-attachments/assets/116390c2-0eab-48be-bed9-41a27f1ad3a1)

Associamos as chaves com seus respectivos valores, e após salvar podemos dar <i>Deploy</i>

![deploy13](https://github.com/user-attachments/assets/68de5cc5-7587-4c71-8c91-9b4009dca6db)

Nossa função está pronta e devidamente configurada. Podemos testa-la e verificar que a Instância foi criada com sucesso. 

![final14](https://github.com/user-attachments/assets/edf75ee8-3bfc-4bcc-abfc-55db761dd41b)

![FINAL_15](https://github.com/user-attachments/assets/fa210cb5-2b39-4514-be9a-11af299e7426)




