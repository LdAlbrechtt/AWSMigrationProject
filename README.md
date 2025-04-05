# Migração e Modernização AWS - Fast Engineering S/A

Projeto de migração de infraestrutura on-premises para AWS, com modernização para Kubernetes (EKS) e serviços gerenciados.

## 📌 Visão Geral
- **Objetivo:** Migrar aplicação de e-commerce para AWS em 2 fases:
  1. **Lift-and-Shift** 
  2. **Modernização** 

## 🛠️ Tecnologias Utilizadas

### Fase 1 - Lift-and-Shift Inicial 

| Tecnologia             | Função                                                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| AWS MGN                | AWS Migration Service para replicar servidores do ambiente on-premise para instâncias EC2 na VPC.                                   |
| AWS DMS                | AWS Database Migration Service para replicar o banco de dados do ambiente on-premise para o Amazon RDS for MySQL (Single-AZ) na VPC. |
| Amazon EC2             | Hospedagem inicial das aplicações (frontend/backend) dentro dos Auto Scaling Groups, após a migração do on-premise.                  |
| AWS ALB                | Application Load Balancer para distribuir o tráfego de usuários para as instâncias EC2.                                              |
| Amazon RDS for MySQL   | Banco de dados MySQL gerenciado na AWS (Single-AZ), como destino inicial da migração do banco de dados on-premise.                     |
| Amazon VPC             | Rede virtual isolada na AWS onde os recursos da Fase 1 residem.                                                                       |
| AWS Route 53           | Serviço de DNS para direcionar o tráfego dos usuários para o ALB.                                                                    |
| AWS Security Groups    | Firewalls virtuais para controlar o tráfego de entrada e saída das instâncias EC2 e do RDS.                                           |
| Amazon EBS Snapshot    | Criação de snapshots para backups dos volumes de disco das instâncias EC2 e do RDS.                                                  |

### Fase 2 - Modernização 

| Tecnologia                 | Função                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Amazon EKS                 | Serviço Kubernetes gerenciado pela AWS para orquestração e escalabilidade dos containers da aplicação.                                |
| Amazon Aurora              | Banco de dados MySQL compatível, escalável e de alta disponibilidade (Private Subnet) como alvo da migração.                        |
| Amazon ECR                 | Registro de containers Docker privado e seguro para armazenar as imagens das aplicações.                                             |
| Docker                     | Plataforma de containerização para empacotar as aplicações e suas dependências em imagens portáteis.                               |
| AWS Load Balancer Controller | Componente Kubernetes que permite que o EKS gerencie os AWS Application Load Balancers para expor os serviços da aplicação.        |
| AWS ALB                    | Application Load Balancer integrado com o EKS para distribuir o tráfego para os pods da aplicação containerizada.                   |
| Amazon S3                  | Armazenamento de objetos escalável e durável para backups do Amazon Aurora e, potencialmente, arquivos estáticos.                     |
| Amazon CloudFront          | Content Delivery Network (CDN) para entregar conteúdo estático de forma rápida e com baixa latência aos usuários globalmente.         |
| AWS WAF                    | Web Application Firewall para proteger a aplicação contra explorações comuns da web e bots maliciosos, integrado com o CloudFront. |
| AWS CloudWatch             | Serviço de monitoramento e observabilidade para coletar métricas, logs e configurar alarmes para os recursos da AWS.                 |
| AWS CloudTrail             | Serviço que registra a atividade da conta AWS e fornece um histórico de eventos de auditoria.                                      |
| AWS Secrets Manager        | Serviço para armazenar e gerenciar segredos (como credenciais de banco de dados) de forma segura.                                    |
| AWS DMS                    | Serviço de Migração de Banco de Dados utilizado para replicar dados do RDS MySQL para o Amazon Aurora (na Fase 2).                  |


## 📊 Arquitetura

### Fase 1: 
![image](https://github.com/user-attachments/assets/42bb0322-3e4a-4f9c-a724-21a404566a33)
 
### Fase 2: 
![image](https://github.com/user-attachments/assets/5e700830-5309-4f91-8ce1-5ba470cab8f6)


## 🔄 Fluxo de Trabalho 

### **Fase 1 - Preparação e Lift-and-Shift Inicial (4-6 semanas)**

1.  **Infraestrutura Base na AWS:**
    * Verificar e/ou configurar a VPC (Virtual Private Cloud) conforme o diagrama.
    * Garantir a existência e configuração do **Route 53** para gerenciamento de DNS.
    * Configurar o **Security Group** associado ao **AWS ALB** para permitir o tráfego esperado.
    * Provisionar o **AWS ALB** (Application Load Balancer) que será o ponto de entrada inicial.
    * Criar o **EC2 FRONT ASG** (Auto Scaling Group) com as instâncias EC2 rodando a versão atual da aplicação.
    * Configurar o **EC2 BACK ASG** (Auto Scaling Group) (se aplicável e distinto do frontend).
    * Garantir que o **RDS MySQL (Single-AZ)** esteja funcionando e acessível pelas instâncias EC2.
    * Implementar o processo de **EBS SNAPSHOT** para backups do ambiente EC2/RDS.

2.  **Migração Inicial do Tráfego (Cutover Inicial):**
    * Atualizar o registro DNS no **Route 53** para apontar o tráfego dos **Usuários** para o **AWS ALB**.
    * Neste ponto, a aplicação continua rodando nas instâncias EC2 (FRONT e BACK ASGs) e utilizando o RDS MySQL.

### **Fase 2 - Modernização com EKS (7+ semanas - Paralelo à Aplicação Existente)**

1.  **Containerização:**
    * **Dockerizar as aplicações (frontend e APIs):** Criar Dockerfiles e construir as imagens para a nova versão da aplicação.
    * **Push das imagens para Amazon ECR:** Armazenar as imagens Docker no Amazon Elastic Container Registry (ECR).

2.  **Infraestrutura Kubernetes:**
    * **Provisionamento do cluster Amazon EKS:** Criar um cluster EKS na sua VPC, configurando as sub-redes e os nós de trabalho.
    * **Implantação do AWS Load Balancer Controller no EKS:** Este controlador permitirá que o Kubernetes gerencie os ALBs.

3.  **Integração do ALB com o EKS:**
    * Definir recursos de **Ingress** no Kubernetes. O AWS Load Balancer Controller irá provisionar um novo ALB (ou utilizar o ALB existente, dependendo da configuração e estratégia) e configurar as regras para rotear o tráfego para os serviços e pods da nova aplicação no EKS.

4.  **Testes Canários e Migração Gradual:**
    * **Configurar testes canários:** Utilizar os recursos do Ingress Controller (ou outras ferramentas de gerenciamento de tráfego) para direcionar uma pequena porcentagem do tráfego (ex: 5%) que chega ao **CloudFront** e passa pelo **AWS WAF** para a nova versão da aplicação rodando nos pods do EKS (através do ALB associado ao EKS).
    * **Monitoramento:** Utilizar o **CloudWatch** e o **CloudTrail** para monitorar a performance, logs e comportamento da nova versão no EKS, comparando com a versão antiga nas EC2.
    * **Aumento gradual do tráfego:** Se os testes canários forem bem-sucedidos, aumentar gradualmente a porcentagem do tráfego direcionado para o EKS (ex: 10%, 25%, 50%, 100%). Isso pode ser feito ajustando as configurações no Ingress ou utilizando ferramentas de gerenciamento de tráfego mais avançadas.

5.  **Migração do Banco de Dados (Gradual ou Cutover):**
    * **Opção 1 (Gradual):** Utilizar o **AWS DMS** para configurar a replicação contínua do **RDS MySQL (Single-AZ)** para o **Amazon Aurora (Private Subnet)**. Isso permite que ambas as versões da aplicação (EC2 e EKS) possam ler do Aurora durante a transição. A escrita pode ser mais complexa e pode exigir lógica na aplicação para direcionar para o banco correto durante a migração.
    * **Opção 2 (Cutover):** Após a maior parte do tráfego ser direcionada para o EKS, realizar um cutover do banco de dados, interrompendo a escrita no RDS MySQL e direcionando toda a leitura e escrita para o Amazon Aurora.

6.  **Otimizações Finais:**
    * **Configuração de auto-scaling no EKS:** Implementar Horizontal Pod Autoscaler (HPA) e Cluster Autoscaler para ajustar automaticamente o número de pods e nós de trabalho no EKS com base na demanda.
    * **Habilitar backups:** Configurar backups regulares do **Amazon Aurora** para o **Amazon S3**. Considerar backups cross-region conforme mencionado.
    * **Utilizar o Secrets Manager:** Migrar segredos (como credenciais de banco de dados) da configuração da aplicação para o **AWS Secrets Manager** e configurar o acesso para as aplicações rodando no EKS e (eventualmente) nas EC2.
    * **Integrar o CloudFront e WAF com o EKS:** Garantir que o **CloudFront** continua sendo o ponto de entrada e que o **AWS WAF** protege o tráfego antes de chegar ao ALB do EKS. Isso pode envolver atualizar as origens do CloudFront para apontar para o ALB do EKS quando a maior parte do tráfego for migrada.

7.  **Descomissionamento da Infraestrutura Antiga:**
    * Após a migração completa do tráfego para o EKS e do banco de dados para o Aurora, e após um período de monitoramento para garantir a estabilidade, descomissionar os **EC2 FRONT ASG**, **EC2 BACK ASG** e possivelmente o **RDS MySQL (Single-AZ)** (se a migração para Aurora for completa).



## 🔄 Fluxo de Dados

**Fluxo Principal de Requisição HTTP(S):**

* **Antes da Modernização Completa:**
    ```
    Usuário → Route 53 → CloudFront (cache) → AWS WAF → AWS ALB (apontando para EC2 FRONT ASG) → EC2 FRONT ASG → EC2 BACK ASG (se aplicável) → RDS MySQL
    ```
* **Após a Modernização (com Testes Canários/Migração Gradual):**
    ```
    Usuário → Route 53 → CloudFront (cache) → AWS WAF → AWS ALB (apontando para uma combinação de EC2 e Pods no EKS) →
        [ EC2 FRONT ASG → EC2 BACK ASG → RDS MySQL ]
        OU
        [ Pods no EKS → Amazon Aurora ]
    ```
* **Após a Modernização Completa:**
    ```
    Usuário → Route 53 → CloudFront (cache) → AWS WAF → AWS ALB (apontando para Pods no EKS) → Pods no EKS → Amazon Aurora / Amazon S3
    ```

**Fluxo de Upload de Arquivos:**

* `Frontend (rodando no browser do Usuário) → API (inicialmente em EC2, depois em Pods no EKS) → Amazon S3 (com configuração de triggers - ex: AWS Lambda - para processamento)`

**Fluxo de Backup Automático:**

* `Amazon Aurora → AWS Backup (configurado para backups diários ou conforme a política definida)`
* `Amazon S3 → Versionamento do S3 (habilitado para manter um histórico contínuo de alterações nos arquivos)`
* `Amazon EBS (dos EC2s - Fase 1) → Snapshots do EBS (agendados conforme a política definida)`

**Outros Fluxos de Dados Relevantes:**

* **Comunicação Interna no EKS:** `Pods no EKS podem se comunicar entre si através de Services e outras ferramentas de rede do Kubernetes.`
* **Acesso a Segredos:** `Aplicações (em EC2 ou no EKS) podem acessar segredos (como credenciais de banco de dados) de forma segura através do AWS Secrets Manager.`
* **Monitoramento:** `Métricas e logs de todos os componentes (CloudFront, WAF, ALB, EC2, EKS, Aurora, S3) são enviados para o Amazon CloudWatch.`
* **Auditoria:** `Todas as ações realizadas na conta AWS são registradas e podem ser visualizadas no AWS CloudTrail.`


## Autores: 
- Lucas Albrecht
- Lucas Ribeiro 
