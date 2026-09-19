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

## Print 1

<img width="1920" height="967" alt="01-terraform-apply" src="https://github.com/user-attachments/assets/eb00d6e1-e252-404b-94be-238239c5041d" />

## Print 2 

<img width="1920" height="967" alt="02-api-produtos" src="https://github.com/user-attachments/assets/ab2c1a43-c97f-4e76-9e7c-5aa452a6c59b" />

## Print 3

<img width="1920" height="967" alt="03-pedido-sqs" src="https://github.com/user-attachments/assets/2877975d-183a-491b-8387-b1e5039070d3" />

## Print 4

<img width="1920" height="967" alt="04-cloudwatch-lambda" src="https://github.com/user-attachments/assets/23d3958b-7326-425f-9d98-e39e794b5d50" />

- [ ]  log da instância EC2 com a aplicação rodando respondendo ao `curl /produtos`.
- [ ]  AWS mostrando a mensagem chegando na fila do SQS.
- [ ]  Lambda executada com sucesso no CloudWatch.

---

## 🛑| Encerramento e Limpeza

Para evitar custos desnecessários e limpar todos os recursos criados no laboratório, execute:
```bash
terraform destroy -auto-approve
```
