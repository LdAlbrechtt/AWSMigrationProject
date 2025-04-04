# Migração e Modernização AWS - Fast Engineering S/A

Projeto de migração de infraestrutura on-premises para AWS, com modernização para Kubernetes (EKS) e serviços gerenciados.

## 📌 Visão Geral
- **Objetivo:** Migrar aplicação de e-commerce para AWS em 2 fases:
  1. **Lift-and-Shift** (migração rápida "as-is")
  2. **Modernização** (Kubernetes + Aurora MySQL)
- **Resultado Esperado:** Infraestrutura escalável, segura e com custo otimizado.

## 🛠️ Tecnologias Utilizadas
### Fase 1 - Lift-and-Shift
| Tecnologia          | Função                                                                 |
|---------------------|-----------------------------------------------------------------------|
| AWS MGN             | Migração de servidores físicos/virtuais para EC2                      |
| AWS DMS             | Replicação do banco MySQL para RDS                                    |
| EC2 (t3.medium/xlarge) | Hospedagem temporária das aplicações (frontend/backend)            |
| ALB                 | Substitui o Nginx como balanceador de carga                           |
| RDS                 | Banco de dados                                                        |


### Fase 2 - Modernização
| Tecnologia          | Função                                                                 |
|---------------------|-----------------------------------------------------------------------|
| EKS                 | Orquestração de containers (Kubernetes gerenciado)                    |
| Aurora MySQL        | Banco de dados Multi-AZ com replicação automática                     |
| S3 + CloudFront     | Armazenamento de arquivos estáticos com CDN                           |
| WAF + Shield        | Proteção contra ataques DDoS e vulnerabilidades OWASP                 |

## 📊 Arquitetura


## 🔄 Fluxo de Trabalho

### **Fase 1 - Lift-and-Shift (4-6 semanas)**
1. **Migração de Servidores:**
   - Uso do AWS MGN para replicar servidores on-premises para EC2
   - Configuração de Security Groups e redes VPC

2. **Migração do Banco de Dados:**
   - AWS DMS para replicação contínua (CDC) do MySQL para RDS
   - Validação de consistência dos dados

3. **Cutover:**
   - Atualização do DNS (Route 53) para apontar para o ALB na AWS
   - Descomissionamento dos servidores on-premises

### **Fase 2 - Modernização (7 semanas)**
1. **Containerização:**
   - Dockerização das aplicações (frontend e APIs)
   - Push das imagens para Amazon ECR

2. **Infraestrutura Kubernetes:**
   - Provisionamento do cluster EKS
   - Configuração do Ingress Controller (ALB)

3. **Migração Gradual:**
   - Testes canários (5% → 100% do tráfego)
   - Monitoramento com CloudWatch e X-Ray

4. **Otimizações Finais:**
   - Configuração de auto-scaling no EKS
   - Habilitar backups cross-region no Aurora

## 🌐 Arquitetura Final Pós-Projeto (Estado Alvo)

### **Diagrama da Solução Modernizada**



## 🔄 Fluxo de Dados

- **Requisição HTTP(S):**  
  `Usuário → CloudFront (cache) → ALB → Pods no EKS → Aurora/S3`  

- **Upload de Arquivos:**  
  `Frontend → API → S3 (com triggers para processamento)`  

- **Backup Automático:**  
  `Aurora → AWS Backup (diário) + S3 Versioning (contínuo)`
