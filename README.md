# ☁️| Projeto E-commerce AWS - Capacita iRede (IFCE) 

Trabalho final desenvolvido para o **Curso de Computação em Nuvem** do Instituto Federal de Educação, Ciência e Tecnologia do Ceará (IFCE) em parceria com o Capacita iRede.

Este projeto implementa uma arquitetura de e-commerce resiliente e assíncrona na AWS utilizando conceitos de Infraestrutura como Código (IaC) e Computação Serverless.

## 🏗️| Arquitetura do Projeto

O fluxo de dados da aplicação segue o modelo desacoplado:
**Usuário** ➔ **API (Flask rodando em EC2)** ➔ **Fila SQS** ➔ **Função Lambda** ➔ **Logs do CloudWatch**

1. O usuário interage com a API Flask hospedada em uma instância EC2.
2. A rota de pedidos envia as mensagens para uma fila do AWS SQS de forma assíncrona.
3. O SQS engatilha uma função AWS Lambda que processa o pedido.
4. O resultado do processamento é registrado e monitorado através do Amazon CloudWatch Logs.

---

## 🗂️| Estrutura do Repositório

```text
├── terraform/     # Infraestrutura como Código (VPC, Subnet, IGW, Route Table, SG, EC2, SQS, Lambda, IAM)
├── app/           # Código do servidor web
│   └── app.py     # API Flask com as rotas `/produtos` e `/pedidos`
└── lambda/        # Código serverless
    └── index.py   # Função Lambda disparada automaticamente pelo SQS
```

---

## 🖥️| Pré-requisitos

Antes de começar, você precisará ter instalado em sua máquina:
* **Terraform 1.0+**
* **AWS CLI** configurada localmente com as credenciais ativas do **AWS Academy Learner Lab**.

---

## 💻| Como Executar

### 1. Implantar a Infraestrutura
Navegue até a pasta do Terraform, inicialize o provedor e aplique as configurações:
```bash
cd terraform
terraform init
terraform apply -auto-approve
```
_Nota: Após a conclusão do deploy, o endereço IP público da sua instância será exibido no terminal através do output `ec2_public_ip`._

### 2. Testar o Fluxo de Pedidos

* **Listar produtos disponíveis (GET):**
  ```bash
  curl http://<IP_EC2>/produtos
  ```

* **Criar um novo pedido (POST - Envia para o SQS):**
  ```bash
  curl -X POST http://<IP_EC2>/pedidos \
       -H "Content-Type: application/json" \
       -d '{"id":1,"produto":"Teclado"}'
  ```

* **Verificar o Processamento:**
  Acesse o Console AWS e verifique os logs gerados no **CloudWatch Logs** sob o grupo: `/aws/lambda/ecommerce-capacita-processor`.

---

## 📉| Diagrama, de Funcionamento
## 
<img width="2172" height="724" alt="Diagrama" src="https://github.com/user-attachments/assets/40bc35ac-6569-4a81-ae83-798ef70613ab" />

## | Evidências do Sistema rodando no LinuxPopOS, e Servidores AWS

## Print 1: Saída do terraform apply — provisionamento da infraestrutura via IaC

Captura do terminal Pop!_OS mostrando o Terraform aplicando 16 recursos na AWS (VPC, Subnet, IGW, Route Table, Security Group, EC2, SQS, Lambda, IAM Roles, Instance Profile, Event Source Mapping). A saída termina com a mensagem Apply complete! Resources: 1 added, 0 changed, 0 destroyed. e os dois outputs gerados:

ec2_public_ip = "3.80.231.255" (IP público da instância)

sqs_queue_url = "https://sqs.us-east-1.amazonaws.com/748222046826/pedidos-a-processar" (URL da fila)

<img width="1920" height="967" alt="4" src="https://github.com/user-attachments/assets/9ab8964b-c540-470d-9657-4fd856c615df" />


## Print 2 : API Flask em execução na EC2 — rota GET /produtos

Requisição HTTP curl http://3.80.231.255/produtos disparada do Pop!_OS, retornando o JSON com a lista de produtos cadastrados:

<img width="1920" height="967" alt="02-api-produtos" src="https://github.com/user-attachments/assets/ab2c1a43-c97f-4e76-9e7c-5aa452a6c59b" />

## Print 3: Requisição curl -X POST http://3.80.231.255/pedidos com payload {"id":1,"produto":"Teclado"}, retornando {"message_id":"0702003d-4426-4185-adfb-ee4b558b70e5","status":"pedido enviado"}. Complementado pela captura do console AWS SQS mostrando a fila pedidos-a-processar criada com sucesso em us-east-1, seu ARN e a política de criptografia SSE-SQS habilitada. Evidencia o padrão desacoplamento via mensageria.

<img width="1920" height="967" alt="Screenshot_2026-09-19_16-06-50" src="https://github.com/user-attachments/assets/94b0824f-e0b1-4707-b05f-4c41f519db76" />

## Print 4: rocessamento serverless — Log da Lambda no CloudWatch

Console AWS CloudWatch exibindo o Log Group /aws/lambda/ecommerce-capacita-processor e um log stream com o processamento completo do pedido pela função Lambda:

text
START RequestId: 0a2ccfcb-e743-523f-9243-ace338dcdb69 Version: $LATEST
[PROCESSADOR DE PEDIDOS] Pedido recebido com sucesso: {'pedido_id': 1, 'produto': 'Teclado', 'status': 'pendente'}
END RequestId: 0a2ccfcb-e743-523f-9243-ace338dcdb69
REPORT RequestId: 0a2ccfcb-... Duration: 1.54 ms Billed Duration: 76 ms
Demonstra o fluxo completo: SQS → Lambda → CloudWatch Logs funcionando de ponta a ponta.

<img width="1600" height="671" alt="44444" src="https://github.com/user-attachments/assets/cef450c5-af33-4a6d-b411-5da6e101fb6e" />


- [ ]  log da instância EC2 com a aplicação rodando respondendo ao `curl /produtos`.
- [ ]  AWS mostrando a mensagem chegando na fila do SQS.
- [ ]  Lambda executada com sucesso no CloudWatch.

---

## 🛑| Encerramento e Limpeza

Para evitar custos desnecessários e limpar todos os recursos criados no laboratório, execute:
```bash
terraform destroy -auto-approve
```
