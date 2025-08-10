# README — Deploy Terraform (S3 frontend, ECS backend, Lambda rotinas)

## Visão geral
Este repositório contém um projeto Terraform que cria:

- Um bucket S3 para hospedar o frontend estático (arquivos HTML/CSS/JS).
- Uma imagem Docker de backend empurrada para ECR e executada em ECS Fargate (código Express).
- Uma função Lambda que grava um arquivo diário no S3.
- Outputs com as URLs/recursos criados.
- Arquivo principal de infraestrutura Terraform que descreve todos os recursos.

> Nota: o Terraform usa um `random_string` para sufixar nomes e o `variable "aws_region"` com `default = "sa-east-1"`.

---

## Pré-requisitos (para o avaliador/testador)
- `terraform` >= 1.0 instalado.
- `aws` CLI configurado (v2 recomendado).
- `docker` instalado e em execução.
- Credenciais AWS com permissões suficientes para criar S3, ECR, ECS, Lambda, IAM e CloudWatch Events.
- Acesso à internet para fazer `docker push` para ECR.

---

## Passo-a-passo para reproduzir o teste

### 1) Configurar credenciais AWS
```bash
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_DEFAULT_REGION=sa-east-1
```

### 2) Preparar a função Lambda (zip)
```bash
mkdir -p lambda
cp "rotina js lambda.js" lambda/index.js
cd lambda
zip rotina.zip index.js
cd ..
```

### 3) Confirmar backend (Docker + ECR)
```bash
cd backend
docker build -t meu-backend:local -f Dockerfile .
docker run -p 3000:3000 meu-backend:local
```

### 4) Executar Terraform
```bash
terraform init
terraform plan -out=tfplan
terraform apply "tfplan"
```

### 5) Verificar outputs
- `frontend_url` → URL do site S3.
- `backend_url` → URL do backend.
- `rotina_bucket` → nome do bucket onde o Lambda grava arquivos.

---

## Testes rápidos após deploy
- Acesse `frontend_url` para ver a página To-Do.
- Faça `curl` ao endpoint do backend (`/` e `/health`).
- Confira o bucket `rotina_bucket` para ver arquivos gerados pela Lambda.

---

## Cleanup
```bash
terraform destroy -auto-approve
```

