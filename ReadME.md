# Documentação do Projeto de Migração e Modernização da Fast Engineering S/A

## Índice
1. [Visão Geral](#visão-geral)
2. [Infraestrutura Atual](#infraestrutura-atual)
3. [Etapa 1: Migração "Lift-and-Shift" (As-Is)](#etapa-1-migração-lift-and-shift-as-is)
    - [O que é "Lift-and-Shift"?](#o-que-é-lift-and-shift)
    - [O que será utilizado?](#o-que-será-utilizado)
    - [Pré-requisitos](#pré-requisitos)
    - [Atividades Necessárias](#atividades-necessárias)
    - [Estimativa dos custos da Migração](#estimativa-dos-custos-da-migração)
5. [Pós migração](#pós-migração)
6. [Etapa 2: Modernização Kubernetes](#etapa-2-modernização-kubernetes)
    - [Atividades Necessárias para a Migração](#atividades-necessárias-para-a-migração)
    - [Ferramentas](#ferramentas)
7. [Separação de Ambientes](#separação-de-ambientes)
8. [Modernização: Explicação das Ferramentas](#modernização-explicação-das-ferramentas)
9. [Conlusão](#conclusão)


## Visão Geral

A empresa Fast Engineering S/A está passando por um crescimento acelerado em seu eCommerce, resultando em um aumento significativo no volume de acessos e transações. A infraestrutura atual não consegue mais atender essa demanda de forma eficiente, impactando a escalabilidade e o desempenho da plataforma. Para solucionar esse desafio, será implementada uma nova arquitetura baseada em AWS, garantindo alta disponibilidade, segurança e performance. O processo será conduzido em duas etapas: primeiro, uma migração Lift-and-Shift para a nuvem, seguida por uma modernização completa utilizando Kubernetes e serviços gerenciados.

## Infraestrutura Atual

A infraestrutura atual é composta por três servidores, cada um com funções específicas dentro da arquitetura da aplicação:

- **Banco de Dados**: Servidor dedicado ao MySQL, armazenando aproximadamente 500GB de dados, com 10GB de RAM e 3 vCPUs.
- **Frontend**: Servidor responsável pela aplicação React, utilizando 5GB de armazenamento, 2GB de RAM e 1 vCPU.
- **Backend**: Servidor que hospeda 3 APIs, além de atuar como balanceador de carga via Nginx e armazenar arquivos estáticos, como imagens e links. Possui 5GB de armazenamento, 4GB de RAM e 2 Core CPU.

Essa estrutura enfrenta desafios de escalabilidade e desempenho, sendo necessária uma migração para a AWS para garantir maior eficiência e disponibilidade.

## Etapa 1: Migração "Lift-and-Shift" (As-Is)

A migração "Lift-and-Shift", também conhecida como "rehosting" ou "as-is", é uma estratégia de migração para a nuvem que envolve mover aplicativos e dados de um ambiente local para a nuvem com o mínimo de alterações possível. O objetivo principal é replicar a infraestrutura existente na nuvem, mantendo a mesma arquitetura e configuração.

### O que é "Lift-and-Shift"?

Imagine que você está mudando de casa. Em vez de reformar completamente a nova casa, você simplesmente pega seus móveis e os coloca nos mesmos lugares que estavam na casa antiga. Isso é essencialmente o que o "Lift-and-Shift" faz com a sua infraestrutura de TI.

### O que será utilizado?


| Ícone | Nome | Descrição |
|------------|------------------------------|------------------------------|
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Migration-Transfer/64/Arch_AWS-Application-Migration-Service_64@5x.png" width="50"> | **AWS Application Migration Service** | Migração de aplicações on-premises para a AWS |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRULf2JOHbvkPux8pEzQrkH70TVSpfgRMzgQA&s" width="50"> | **Amazon EC2** | Instâncias de servidores virtuais na AWS |
| <img src="https://cdn.worldvectorlogo.com/logos/amazon-s3-simple-storage-service.svg" width="50"> | **Amazon S3** | Armazenamento escalável de objetos na nuvem |
| <img src="https://cdn.worldvectorlogo.com/logos/amazon-elastic-block-store-1.svg" width="50"> | **Amazon Elastic Block Store** | Armazenamento em bloco |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7L7fI-Ozxh2ni9T2E7rgX_CU-VNMOpoXfwpIxYIaifUcJL_NQ0ZJi8mGHWNRdiFXmres&usqp=CAU" width="50"> | **Amazon RDS** | Banco de dados relacional gerenciado |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Database/64/Arch_AWS-Database-Migration-Service_64@5x.png" width="50"> | **AWS Database Migration Service** | Migração de bancos de dados para a AWS |

### Pré-requisitos

- **Sub-rede da Área de Teste**: Antes de configurar o Application Migration Service, crie uma sub-rede para ser usada como área de preparação para dados replicados dos seus servidores de origem para AWS (ou seja, um plano de dados). Você deve especificar essa sub-rede no modelo de Configurações de Replicação ao acessar pela primeira vez o console do Application Migration Service.
- **Requisitos de Rede**:
    - Os servidores de replicação que são iniciados pelo Serviço de Migração de Aplicativos na sub-rede da sua área de armazenamento precisam ser capazes de enviar dados para o endpoint da API do Serviço de Migração de Aplicativos em `https://mgn.<region>.amazonaws.com/`, onde `<region>` é o código da região da AWS para a qual você está replicando (por exemplo, se sua aplicação está no norte da virgina: `https://mgn.us-east-1.amazonaws.com`).
    - O instalador do AWS Replication Agent deve ter acesso à URL do bucket do Amazon S3 da região da AWS que você está usando com o Application Migration Service.
    - A sub-rede da área de armazenamento deve ter acesso ao Amazon S3.
    - Os servidores de origem nos quais o Agente de AWS Replicação está instalado devem ser capazes de enviar dados para os servidores de replicação na sub-rede da área de armazenamento e para o endpoint da API do Application Migration Service em `https://mgn.<region>.amazonaws.com/`.

### Atividades Necessárias

#### Migração "Lift-and-Shift": Passo a Passo

1. **Preparação do Ambiente On-Premise e AWS**:
    - **Criar Sub-rede da Área de Teste**:
        - No console da AWS, navegue até o serviço VPC.
        - Crie uma nova sub-rede dedicada para a área de preparação.
        - Anote o ID da sub-rede, pois você precisará especificá-lo no modelo de Configurações de Replicação.
    - **Instalação do Agente de Replicação AWS**:
        - Em cada servidor local (front01, back01, db01), baixe e instale o AWS Replication Agent.
        - O instalador do agente deve ter acesso à URL do bucket do Amazon S3 da região da AWS que você está usando.
    - **Configuração de Rede**:
        - Certifique-se de que os servidores locais e os servidores de replicação MGN possam se comunicar com os endpoints do AWS MGN na porta 443.
        - Configure regras de segurança no seu VPC para permitir o tráfego necessário.

2. **Configuração dos Endpoints de Interface**:
    - **Endpoint da Interface para o Application Migration Service**:
        - Abra o console da Amazon VPC.
        - No painel de navegação, escolha Endpoints e depois Create Endpoint.
        - Em Service category, escolha AWS Services.
        - Em Service Name, digite `com.amazonaws.<region>.mgn` e selecione Interface como o tipo.
        - Para VPC, selecione a VPC de destino.
        - Em Subnets, selecione as sub-redes onde criar as interfaces de rede do endpoint.
        - Ative o DNS privado para o endpoint da interface.
        - Selecione um grupo de segurança que permita a entrada da sub-rede VPC da área de armazenamento via TCP 443.
        - Escolha Create Endpoint.
    - **Endpoint da Interface para a Amazon EC2**:
        - Siga as mesmas instruções acima, mas em Service Name, digite `com.amazonaws.<region>.ec2`.
    - **Endpoint da Interface para o Amazon S3**:
        - Siga as mesmas instruções acima, mas em Service Name, digite `com.amazonaws.<region>.s3`.
        - Desmarque Habilitar nome DNS, pois o DNS privado não é compatível com endpoints da interface do Amazon S3.
    - **Endpoint do Gateway do Amazon S3**:
        - Crie um endpoint de gateway Amazon S3 na VPC à qual a sub-rede da área de teste pertence.
        - Atualize as tabelas de rotas da sub-rede de teste com as rotas relevantes.

3. **Replicação e Migração dos Servidores**:
    - **Replicação Contínua com MGN**:
        - Estabeleça a conexão entre os servidores locais e os servidores de replicação via AWS Direct Connect ou VPN.
        - Os dados são criptografados em trânsito usando criptografia AES de 256 bits.
        - Volumes Elastic Block Store (EBS) são criados com o mesmo tamanho dos discos originais, mantendo os dados sincronizados na AWS.
    - **Migração do Banco de Dados com DMS**:
        - Utilize o AWS Database Migration Service (DMS) para migrar o banco de dados MySQL para o Amazon RDS.
        - Configure o DMS para replicar os dados do banco de dados de origem para o RDS.
    - **Snapshots e Conectividade com API EC2**:
        - O servidor de replicação realizará chamadas de API para criar snapshots dos volumes EBS de preparação durante a replicação.
        - Certifique-se de que a conectividade com o endpoint da API EC2 na porta 443 esteja ativa.

4. **Testes**:
    - **Instâncias de Teste**:
        - Inicie instâncias EC2 na sub-rede de área de testes com base no modelo de execução configurado.
        - Essas instâncias se conectarão a cópias atualizadas dos volumes EBS da área de preparação.
        - Importante: essas cópias não serão mais sincronizadas com os servidores locais.

![Migração](./img/diagramas/migracao.gif)

## Estimativa dos custos da Migração:

| Ícone | Nome | Custo (USD) | Custo 3 dias (USD) |
|------------|------------------------------|-------------|-------------------|
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRULf2JOHbvkPux8pEzQrkH70TVSpfgRMzgQA&s" width="50"> | **Amazon EC2** | 6,13 | 0,61 |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRULf2JOHbvkPux8pEzQrkH70TVSpfgRMzgQA&s" width="50"> | **Amazon EC2** | 10,99 | 1,10 |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7L7fI-Ozxh2ni9T2E7rgX_CU-VNMOpoXfwpIxYIaifUcJL_NQ0ZJi8mGHWNRdiFXmres&usqp=CAU" width="50"> | **Amazon RDS for MySQL** | 396,95 | 39,70 |
| <img src="https://cdn.worldvectorlogo.com/logos/amazon-s3-simple-storage-service.svg" width="50"> | **Amazon S3** | 1,15 | 0,12 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Database/64/Arch_AWS-Database-Migration-Service_64@5x.png" width="50"> | **AWS Database Migration Service** | 507,91 | 50,79 |
| <img src="https://github.com/user-attachments/assets/74224709-c040-46dd-858c-6307886fb90d" width="50"> | **Amazon VPC** | 10,95 | 1,10 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Migration-Transfer/64/Arch_AWS-Application-Migration-Service_64@5x.png" width="50"> | **AWS Application Migration Service** | 0,00 | 0,00 |

[Link para estimativa de Custos da Migração](https://calculator.aws/#/estimate?id=98c123ba0dd4c1b98e488fcb8f3600ee4882f41d)

(934,08 / 30) * 3  = 93,40 USD

![Custo Migração](./img/estimativa/estimativaMigracao.png)

## Pós migração:

Após fazer a migração e realizar todos os testes, a estrutura utilizará as seguintes ferramentas da AWS:
| Ícone | Nome | Descrição |
|---|---|---|
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-Route-53_64@5x.png" width="50"> | Route 53 | Serviço de DNS (Sistema de Nomes de Domínio). |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Security-Identity-Compliance/64/Arch_AWS-WAF_64.png" width="50"> | WAF (Web Application Firewall) | Firewall de aplicações web. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-CloudFront_64.png" width="50"> | CloudFront | Serviço de rede de entrega de conteúdo (CDN). |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Elastic-Load-Balancing_Application-Load-Balancer_48_Light.png" width="50"> | Load Balancer | Distribuidor de tráfego. |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Amazon-VPC_NAT-Gateway_48_Light.png" width="50"> | NAT (Network Address Translation) | Serviço de tradução de endereços de rede. |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7L7fI-Ozxh2ni9T2E7rgX_CU-VNMOpoXfwpIxYIaifUcJL_NQ0ZJi8mGHWNRdiFXmres&usqp=CAU" width="50"> | RDS (Relational Database Service) | Serviço de banco de dados relacional gerenciado. |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRULf2JOHbvkPux8pEzQrkH70TVSpfgRMzgQA&s" width="50"> | Amazon EC2 | Instâncias de servidores virtuais na AWS |

### Diagrama após a migração

![PósMigração](./img/diagramas/Posmigracao.png)

### Valores da Infra da Pos Migração

[Link para estimativa de Custos da Pós Migração](https://calculator.aws/#/estimate?id=e55363d2fe38987bf0ba08693e51455473e227d7)

![estimativaPosMigracao](./img/estimativa/estimativaPosMigracao.png)

| Ícone | Nome | Custo Mensal (USD) |
|------------|------------------------------|-------------|
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRULf2JOHbvkPux8pEzQrkH70TVSpfgRMzgQA&s" width="50"> | **Amazon EC2 (Front-End)** | 12,26 |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRULf2JOHbvkPux8pEzQrkH70TVSpfgRMzgQA&s" width="50"> | **Amazon EC2 (Back-End)** | 24,38 |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7L7fI-Ozxh2ni9T2E7rgX_CU-VNMOpoXfwpIxYIaifUcJL_NQ0ZJi8mGHWNRdiFXmres&usqp=CAU" width="50"> | **Amazon RDS for MySQL** | 706,30 |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Elastic-Load-Balancing_Application-Load-Balancer_48_Light.png" width="50"> | **Load Balancer** | 33,32 | 
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Security-Identity-Compliance/64/Arch_AWS-WAF_64@5x.png" width="50"> | **AWS Web Application Firewall (WAF)** | 0,00 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-CloudFront_64@5x.png" width="50"> | **Amazon CloudFront** | 0,00 |
| <img src="https://github.com/user-attachments/assets/74224709-c040-46dd-858c-6307886fb90d" width="50"> | **Amazon Virtual Private Cloud (VPC)** | 74,70 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-Route-53_64@5x.png" width="50"> | **Amazon Route 53** | 15,50 |



## Etapa 2: Modernização Kubernetes

### Atividades Necessárias para a Migração

- Reestruturação para um ambiente Kubernetes na AWS (EKS).
- Armazenamento de monitoramento com AWS CloudWatch e Prometheus.
- Segurança aprimorada com AWS WAF e Secrets Manager.

### Ferramentas da AWS:

| Ícone | Nome | Descrição |
|---|---|---|
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Containers/64/Arch_Amazon-Elastic-Container-Registry_64.png" width="50"> | Amazon ECR (Elastic Container Registry) | Registro de imagens Docker. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Containers/64/Arch_Amazon-Elastic-Kubernetes-Service_64.png" width="50"> | Amazon EKS (Elastic Kubernetes Service) | Serviço gerenciado de Kubernetes. |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7L7fI-Ozxh2ni9T2E7rgX_CU-VNMOpoXfwpIxYIaifUcJL_NQ0ZJi8mGHWNRdiFXmres&usqp=CAU" width="50"> | Amazon RDS (Relational Database Service) | Serviço de banco de dados relacional gerenciado. |
| <img src="https://cdn.worldvectorlogo.com/logos/amazon-s3-simple-storage-service.svg" width="50"> | Amazon S3 (Simple Storage Service) | Serviço de armazenamento de objetos. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Management-Governance/64/Arch_Amazon-CloudWatch_64.png" width="50"> | CloudWatch | Serviço de monitoramento e observabilidade. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-Route-53_64@5x.png" width="50"> | Route 53 | Serviço de DNS (Sistema de Nomes de Domínio). |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Security-Identity-Compliance/64/Arch_AWS-WAF_64.png" width="50"> | WAF (Web Application Firewall) | Firewall de aplicações web. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-CloudFront_64.png" width="50"> | CloudFront | Serviço de rede de entrega de conteúdo (CDN). |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Amazon-VPC_Internet-Gateway_48_Light.png" width="50"> | Internet Gateway | Componente de rede para comunicação com a Internet. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Security-Identity-Compliance/64/Arch_AWS-Identity-and-Access-Management_64.png" width="50"> | IAM (Identity and Access Management) | Serviço de gerenciamento de identidade e acesso. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Security-Identity-Compliance/64/Arch_AWS-Secrets-Manager_64@5x.png" width="50"> | Secrets Manager | Serviço para gerenciamento de segredos. |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Elastic-Load-Balancing_Application-Load-Balancer_48_Light.png" width="50"> | Load Balancer | Distribuidor de tráfego. |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Amazon-VPC_NAT-Gateway_48_Light.png" width="50"> | NAT (Network Address Translation) | Serviço de tradução de endereços de rede. |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Storage/64/Arch_AWS-Backup_64@5x.png" width="50"> | Backup | Serviço de backup. |

## Separação de Ambientes

Decidimos criar três ambientes de infraestrutura: Desenvolvimento, Homologação e Produção. Cada ambiente possui suas próprias branches no GitLab, com testes manuais e automatizados, e acessos separados por políticas de segurança.

- **Ambiente de Desenvolvimento**:
  - Utiliza a branch `dev` no GitLab.
  - Focado em desenvolvimento e testes iniciais.
  - Acesso restrito aos desenvolvedores.
  - Testes automatizados são executados para garantir a qualidade do código.

![AmbienteDeDev](./img/diagramas/EKSDev.drawio.png)
![estimativaEKSDev](./img/estimativa/estimativaDevEKS.png)

- **Ambiente de Homologação**:
  - Utiliza a branch `homolog` no GitLab.
  - Ambiente que se aproxima do produto final.
  - Acesso restrito a desenvolvedores e equipe de QA.
  - Transição da branch `dev` para `homolog` é feita manualmente através de pull requests aprovados.
  - Testes manuais e automatizados são realizados para validar as funcionalidades.

![AmbienteDeHomologação](./img/diagramas/EKSHomolog.drawio.png)
![estimativaEKSHomologacao](./img/estimativa/estimativaEKSHomologacao.png)

- **Ambiente de Produção**:
  - Utiliza a branch `main` no GitLab.
  - Ambiente que está em produção e acessível aos usuários finais.
  - Acesso restrito à equipe de operações.
  - Transição da branch `homolog` para `main` é feita após aprovação final.
  - Monitoramento contínuo e testes de desempenho são realizados para garantir a estabilidade.

![AmbienteDeProd](./img/diagramas/EKSProd.drawio.png)
[Link para estimativa de Custos da Modernização](https://calculator.aws/#/estimate?id=4ca7450494d03d50d2304178975068e37db647e3)
![estimativaEKSProd](./img/estimativa/estimativaEKSProd.png)

| Ícone | Nome | Custo mensal (USD) |
|------------|------------------------------|-------------|
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Compute/64/Arch_Amazon-EC2_64.png" width="50"> | **Amazon EC2 (Front-End)** | 15,04 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Compute/64/Arch_Amazon-EC2_64.png" width="50"> | **Amazon EC2 (Back-End)** | 27,45 |
| <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7L7fI-Ozxh2ni9T2E7rgX_CU-VNMOpoXfwpIxYIaifUcJL_NQ0ZJi8mGHWNRdiFXmres&usqp=CAU" width="50"> | **Amazon RDS for MySQL** | 6.030,92 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Containers/64/Arch_Amazon-Elastic-Kubernetes-Service_64.png" width="50"> | **Amazon EKS** | 73,00 |
| <img src="https://cdn.worldvectorlogo.com/logos/amazon-s3-simple-storage-service.svg" width="50"> | **Amazon S3** | 23,55 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Containers/64/Arch_Amazon-Elastic-Container-Registry_64.png" width="50"> | **Amazon Elastic Container Registry (ECR)** | 5,00 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-Route-53_64.png" width="50"> | **Amazon Route 53** | 3,00 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Security-Identity-Compliance/64/Arch_AWS-WAF_64.png" width="50"> | **AWS Web Application Firewall (WAF)** | 10,00 |
| <img src="https://cloud-icons.onemodel.app/aws/Architecture-Service-Icons_01312023/Arch_Networking-Content-Delivery/64/Arch_Amazon-CloudFront_64.png" width="50"> | **Amazon CloudFront** | 0,00 |
| <img src="https://cloud-icons.onemodel.app/aws/Resource-Icons_01312023/Res_Networking-and-Content-Delivery/Res_48_Light/Res_Amazon-VPC_Internet-Gateway_48_Light.png" width="50"> | **Amazon Virtual Private Cloud (VPC)** | 65,88 |





## Modernização: Explicação das Ferramentas

1. **Amazon ECR (Elastic Container Registry)**:
    - **O que faz**: É um serviço de registro de contêineres Docker totalmente gerenciado.
    - **Importância**: Permite armazenar e gerenciar imagens Docker, que são usadas para empacotar e distribuir a aplicação em contêineres.

2. **Amazon EKS (Elastic Kubernetes Service)**:
    - **O que faz**: É um serviço gerenciado de Kubernetes que facilita a implantação, o gerenciamento e o escalonamento de aplicações em contêineres.
    - **Importância**: Permite orquestrar contêineres de forma eficiente, garantindo alta disponibilidade, escalabilidade e resiliência da aplicação.

3. **Amazon RDS (Relational Database Service)**:
    - **O que faz**: É um serviço de banco de dados relacional gerenciado que oferece suporte a vários mecanismos de banco de dados, como MySQL, PostgreSQL e Aurora.
    - **Importância**: Fornece um banco de dados gerenciado, eliminando a necessidade de gerenciar a infraestrutura subjacente e garantindo alta disponibilidade e escalabilidade.

4. **Amazon S3 (Simple Storage Service)**:
    - **O que faz**: É um serviço de armazenamento de objetos altamente escalável e durável.
    - **Importância**: Permite armazenar e recuperar qualquer tipo de dado, como arquivos estáticos, backups e logs.

5. **CloudWatch**:
    - **O que faz**: É um serviço de monitoramento e observabilidade que coleta métricas, logs e eventos de recursos AWS e aplicações.
    - **Importância**: Permite monitorar a saúde e o desempenho da aplicação, identificar problemas e tomar decisões informadas.

6. **Route 53**:
    - **O que faz**: É um serviço de DNS (Sistema de Nomes de Domínio) altamente disponível e escalável.
    - **Importância**: Permite gerenciar nomes de domínio e rotear o tráfego para a aplicação na AWS.

7. **WAF (Web Application Firewall)**:
    - **O que faz**: É um firewall de aplicações web que protege a aplicação contra ataques comuns, como injeção de SQL e cross-site scripting.
    - **Importância**: Garante a segurança da aplicação, protegendo-a contra ameaças e vulnerabilidades.

8. **CloudFront**:
    - **O que faz**: É um serviço de rede de entrega de conteúdo (CDN) que acelera a entrega de conteúdo para usuários em todo o mundo.
    - **Importância**: Melhora o desempenho da aplicação, reduzindo a latência e o tempo de carregamento para usuários geograficamente dispersos.

9. **Internet Gateway**:
    - **O que faz**: É um componente de rede que permite a comunicação entre a VPC e a Internet.
    - **Importância**: Permite que a aplicação acesse a Internet e que os usuários acessem a aplicação a partir da Internet.

10. **IAM (Identity and Access Management)**:
    - **O que faz**: É um serviço de gerenciamento de identidade e acesso que permite controlar o acesso a recursos AWS.
    - **Importância**: Garante que **CodePipeline, EKS, RDS, S3** e outros serviços só possam ser acessados por quem tem permissão.  

11. **Secrets Manager**:
    - **O que faz**: É um serviço que permite gerenciar segredos, como senhas, chaves de API e certificados.
    - **Importância**: Melhora a segurança da aplicação, evitando que segredos sejam armazenados em código ou em arquivos de configuração.

12. **Load Balancer**:
    - **O que faz**: Distribui o tráfego de entrada entre várias instâncias EC2 ou contêineres.
    - **Importância**: Melhora a disponibilidade e a escalabilidade da aplicação, distribuindo a carga de trabalho de forma uniforme.
      
13. **Bastion Host**:
    - **O que faz**: São **servidores intermediários** para SSH dentro da VPC, evitando exposição direta das instâncias privadas à internet.  
    - **Importância**: Os administradores se conectam ao Bastion Host para acessar os nós do **EKS**.  

14. **NAT (Network Address Translation)**:
    - **O que faz**: Permite que instâncias em sub-redes privadas acessem a Internet sem serem acessíveis a partir da Internet.
    - **Importância**: Melhora a segurança da aplicação, ocultando as instâncias privadas da Internet.

15. **Backup**:
    - **O que faz**: É um serviço que permite fazer backup e restaurar dados de recursos AWS.
    - **Importância**: Garante a segurança dos dados, permitindo a recuperação em caso de falha ou desastre.

16. **GitLab**:
    - **O que faz**: É uma plataforma de DevOps que fornece repositórios Git, CI/CD, gerenciamento de projetos, entre outros.
    - **Importância**: Facilita a colaboração entre desenvolvedores, automatiza pipelines de CI/CD e gerencia o ciclo de vida do desenvolvimento de software.

17. **Terraform**:
    - **O que faz**: É uma ferramenta de infraestrutura como código (IaC) que permite definir e provisionar infraestrutura através de arquivos de configuração.
    - **Importância**: Automatiza a criação e o gerenciamento de infraestrutura, garantindo consistência e facilitando a replicação de ambientes.

## Conclusão

A migração e modernização da infraestrutura da Fast Engineering S/A para a AWS representa um marco significativo na trajetória de crescimento e inovação da empresa. Este projeto foi cuidadosamente planejado e executado em duas etapas principais: a migração "Lift-and-Shift" e a modernização com Kubernetes, cada uma com objetivos claros e benefícios distintos.

### Migração "Lift-and-Shift"

A primeira etapa, a migração "Lift-and-Shift", permite que a Fast Engineering S/A mova suas aplicações e dados para a nuvem. Esta abordagem garante uma transição rápida e eficiente, replicando a infraestrutura existente na AWS. Utilizando serviços como o AWS Application Migration Service, Amazon EC2, Amazon S3, e Amazon RDS, a empresa conseguiu manter a continuidade dos negócios enquanto se beneficiava das vantagens da nuvem, como escalabilidade, alta disponibilidade e segurança aprimorada.


### Modernização com Kubernetes

A segunda etapa, a modernização com Kubernetes, foi projetada para transformar a infraestrutura da Fast Engineering S/A em uma plataforma moderna e altamente eficiente. Ao adotar o Amazon EKS (Elastic Kubernetes Service) e outras ferramentas de contêinerização, a empresa conseguiu orquestrar suas aplicações de forma mais eficiente, garantindo alta disponibilidade, escalabilidade e resiliência.

A reestruturação para um ambiente Kubernetes na AWS envolve a implementação de práticas de DevOps, como a automação de pipelines de CI/CD com GitLab e a definição de infraestrutura como código (IaC) com Terraform. Essas práticas não apenas melhoraram a eficiência operacional, mas também garantiram a consistência e a repetibilidade dos ambientes de desenvolvimento, homologação e produção.

### Benefícios e Futuro

Com a nova infraestrutura baseada em serviços gerenciados da AWS, a Fast Engineering S/A estará melhor equipada para lidar com picos de demanda, melhorar a experiência do usuário e reduzir custos operacionais.

Além disso, a separação de ambientes em desenvolvimento, homologação e produção, com políticas de segurança e testes rigorosos, assegura que a empresa possa continuar inovando e lançando novas funcionalidades com confiança.

Este projeto de migração e modernização não apenas resolve os desafios atuais de escalabilidade e desempenho, mas também posiciona a Fast Engineering S/A para um futuro de inovação contínua e crescimento sustentável. A empresa está agora preparada para enfrentar os desafios do mercado e aproveitar as oportunidades de crescimento, com uma infraestrutura robusta, segura e escalável.
