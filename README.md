# 🚀 Criando uma Instância EC2 na AWS via CloudShell

Este guia ensina como criar uma instância EC2 utilizando o **AWS CloudShell** e comandos da **AWS CLI**.

---

## 📦 Pré-requisitos

Antes de começar, certifique-se de:

- Ter uma conta na AWS com permissões para criar EC2, Security Groups e Key Pairs.
- Estar logado no **AWS CloudShell** na região desejada.
- Já ter uma **Key Pair** criada na AWS EC2 (caso não tenha, crie pelo console ou via CLI).

---

## 🛠️ Etapas da Configuração

### 1️⃣ Defina as Variáveis

```bash
  GRUPO_SEGURANCA="nome-do-security-group"
  NOME_INSTANCIA="nome-instancia"
  PAR_CHAVE="sua-chave-ec2"
```

### 2️⃣ Crie o Security Group

```bash
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name $GRUPO_SEGURANCA \
  --description "Permitir HTTP" \
  --query "GroupId" \
  --output text)
```

### 3️⃣ Libere a Porta 80 (HTTP)

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SECURITY_GROUP_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

Este comando cria um Security Group para permitir acesso HTTP (porta 80).

### 4️⃣ Crie a Instância EC2

```bash
aws ec2 run-instances \
  --instance-type t2.micro \
  --image-id $(aws ssm get-parameters-by-path \
    --path "/aws/service/ami-amazon-linux-latest" \
    --query "Parameters[?ends_with(Name, 'al2023-ami-kernel-default-x86_64')].Value" \
    --output text) \
  --security-group-ids $SECURITY_GROUP_ID \
  --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=$NOME_INSTANCIA}]" \
  --key-name $PAR_CHAVE \
  --user-data "#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Olá do seu servidor web!</h1></html>' > /var/www/html/index.html"
```

### ✅ Verificando a Instância
- Vá até o Console da AWS EC2.
- Localize sua instância recém-criada.
- Copie o IPv4 público.
- Acesse no navegador:
```bash
  http://<IP-da-instância>
```
Você verá a mensagem: Olá do seu servidor web!

### 🧹 Limpeza
Para evitar custos desnecessários:

```bash
aws ec2 terminate-instances --instance-ids <ID_DA_INSTANCIA>
aws ec2 delete-security-group --group-id $SECURITY_GROUP_ID
```
