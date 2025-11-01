## AWS CloudFormation

O AWS CloudFormation é um serviço que automatiza a criação, configuração e gerenciamento de recursos da AWS por meio de modelos (templates) definidos em arquivos YAML ou JSON. Ele permite que você gaste menos tempo configurando manualmente serviços e mais tempo desenvolvendo seus aplicativos.

Em vez de criar e conectar manualmente recursos como instâncias EC2, grupos de Auto Scaling, bancos de dados RDS e balanceadores de carga (ELB), o CloudFormation executa todo esse processo automaticamente com base no modelo fornecido.

Cada modelo gera uma stack (pilha) — que é um conjunto de recursos criados, configurados e gerenciados como uma única unidade. Assim, é possível criar, atualizar ou excluir toda a infraestrutura de uma vez, de forma simples e segura.

Além disso, o CloudFormation facilita a replicação de ambientes em várias regiões, garantindo consistência e alta disponibilidade das aplicações. Com isso, o mesmo modelo pode ser reutilizado para implantar os mesmos recursos de forma padronizada em diferentes ambientes (como desenvolvimento, teste e produção).

Outro benefício importante é o controle de versão: como os templates são arquivos de texto, podem ser versionados em sistemas como o Git. Isso permite rastrear alterações na infraestrutura, identificar quem fez cada modificação e até reverter versões anteriores, caso seja necessário.


# Implementando uma infraestrutura automatizada com AWS CloudFormation

Este guia apresenta o passo a passo completo para criar, validar e automatizar a implementação de infraestrutura na AWS utilizando o **CloudFormation**.

---

## ☁️ 1) Planejamento e requisitos

- Defina os recursos que serão provisionados (VPC, subnets, EC2, RDS, S3, IAM, etc.).  
- Garanta que o usuário tenha permissões IAM adequadas para criação e gerenciamento de recursos.

---

## ⚙️ 2) Escrever o template (YAML ou JSON)

Crie um arquivo `template.json` com a estrutura básica:

```
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "Template de exemplo - cria um bucket S3",
  "Parameters": {
    "BucketName": {
      "Type": "String",
      "Description": "Nome do bucket S3"
    }
  },
  "Resources": {
    "MeuBucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": {
          "Ref": "BucketName"
        },
        "AccessControl": "Private"
      }
    }
  },
  "Outputs": {
    "BucketNameOut": {
      "Description": "Nome do bucket criado",
      "Value": {
        "Ref": "MeuBucket"
      }
    }
  }
}

```

---

## ✅ 3) Validar o template localmente

```bash
aws cloudformation validate-template --template-body file://template.json
```

Esse comando verifica erros de sintaxe e estrutura.

---

## 📦 4) Empacotar artefatos (quando houver)

Caso o template referencie código local (por exemplo, funções Lambda), use:

```bash
aws cloudformation package   --template-file template.json   --s3-bucket meu-bucket-de-deploy   --output-template-file template-packaged.json
```

---

## 🚀 5) Criar ou atualizar a *stack*

```bash
aws cloudformation deploy   --template-file template-packaged.json   --stack-name minha-stack-exemplo   --capabilities CAPABILITY_NAMED_IAM   --parameter-overrides BucketName=meu-bucket-exemplo
```

Para atualizar, basta rodar o mesmo comando com as alterações no template.

---

## 🧩 6) Revisar mudanças com *Change Sets*

```bash
aws cloudformation create-change-set   --stack-name minha-stack-exemplo   --change-set-name revisao-2025-11-01   --template-body file://template-packaged.json   --capabilities CAPABILITY_NAMED_IAM
```

Use *change sets* para revisar as mudanças antes da aplicação.

---

## 🪶 7) Monitorar e tratar erros

- Acompanhe o progresso no console AWS ou com:
  ```bash
  aws cloudformation describe-stack-events --stack-name minha-stack-exemplo
  ```
- Em caso de falha, o CloudFormation realiza rollback automático.

---

## 📁 8) Boas práticas

- Modularize templates (use nested stacks).  
- Versione os templates em Git.  
- Use parâmetros e outputs para reuso.  
- Evite incluir segredos no código.  
- Use *Drift Detection* para detectar mudanças fora do template.

---

## 🔄 9) Automatização em CI/CD

Fluxo comum de pipeline:

1. Commit do template no repositório.  
2. Pipeline executa:
   - `cfn-lint` ou `validate-template`
   - Testes de integração
   - `aws cloudformation package`
   - `aws cloudformation deploy`
3. Notificação em caso de erro via SNS, Slack ou e-mail.

---

## ✅ 10) Checklist antes do deploy

- [ ] Validar permissões IAM  
- [ ] Verificar nomes únicos de recursos (S3, etc.)  
- [ ] Testar em ambiente de homologação  
- [ ] Criar plano de rollback  
- [ ] Validar *change set* antes do deploy

---

## 🧠 Conclusão

O AWS CloudFormation permite **automatizar a criação e o gerenciamento de infraestrutura** com segurança, reprodutibilidade e controle de versão.  
Integrando com pipelines CI/CD, é possível atingir **infraestrutura totalmente automatizada e auditável**.
