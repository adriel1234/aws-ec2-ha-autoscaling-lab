# ☁️ Laboratório AWS: Gerenciamento de Instâncias EC2, Auto Scaling e Alta Disponibilidade

Este repositório contém minhas anotações, decisões de arquitetura e aprendizados práticos do laboratório de gerenciamento de instâncias **Amazon EC2** na **AWS**.

O foco da entrega é documentar de forma clara como gerenciar instâncias EC2 com **Alta Disponibilidade (Multi-AZ)**, **Application Load Balancer (ALB)**, **Auto Scaling Group (ASG)** e **AWS Systems Manager (Session Manager)** para acesso seguro sem abrir a porta SSH (`22`).

---

## 📐 Arquitetura da Solução

![Diagrama de Arquitetura AWS no Draw.io](images/arquitetura-drawio.png)

A topologia desenhada no estilo Draw.io contempla:

* **Usuários / Internet:** Acessam a aplicação web pelo endereço DNS do **Application Load Balancer (ALB)**.
* **Application Load Balancer (ALB):** Distribui requisições HTTP entre múltiplas Zonas de Disponibilidade (`us-east-1a` e `us-east-1b`) e verifica a saúde das instâncias por meio de *Health Checks*.
* **Auto Scaling Group (ASG):** Gerencia a quantidade de instâncias (`Min: 2 | Desired: 2 | Max: 4`), substituindo instâncias com falha e escalando automaticamente com política de *Target Tracking* (`CPU > 50%`).
* **Instâncias EC2 (`Web App 01` e `Web App 02`):** Executam a aplicação web em subnets públicas nas duas zonas de disponibilidade.
* **AWS Systems Manager (Session Manager):** Permite conectar às instâncias via IAM Role sem expor a porta `22` (SSH) à internet.

---

## 🛠️ Etapas Realizadas no Laboratório

### 1. Preparação de IAM e Security Groups
* Criação de uma **IAM Role** para as instâncias EC2 com a política `AmazonSSMManagedInstanceCore`, habilitando o acesso via **Session Manager**.
* Criação do Security Group do ALB (`sg-alb`) permitindo tráfego HTTP (porta `80`).
* Criação do Security Group das instâncias EC2 (`sg-ec2-web`) permitindo tráfego na porta `80` apenas a partir do `sg-alb`, mantendo a porta `22` fechada.

### 2. Criação do Launch Template
* Definição da AMI (ex.: *Amazon Linux 2023*), tipo de instância compatível com o nível gratuito (`t2.micro` ou `t3.micro`), IAM Instance Profile e Security Group `sg-ec2-web`.
* Configuração da inicialização da aplicação web no momento do boot da instância via *User Data*.

### 3. Configuração do Target Group e Application Load Balancer
* Criação de um **Target Group** do tipo *Instances* na porta `80` com *Health Check* apontando para `/`.
* Criação de um **Application Load Balancer** (*Internet-facing*) em subnets de duas Availability Zones (`us-east-1a` e `us-east-1b`), associando o listener `HTTP:80` ao *Target Group*.

### 4. Configuração do Auto Scaling Group (ASG)
* Vinculação do *Launch Template*, das duas Availability Zones e do *Target Group*.
* Configuração da capacidade em `Min: 2`, `Desired: 2` e `Max: 4`.
* Configuração da política dinâmica de *Target Tracking* para manter a utilização média de CPU em torno de `50%`.

### 5. Testes e Validação
* **Balanceamento entre zonas:** Atualizando a página pelo DNS do ALB para observar a alternância entre as instâncias em `us-east-1a` e `us-east-1b`.
* **Recuperação automática (*Self-Healing*):** Encerrando manualmente uma instância EC2 e acompanhando o ASG subir uma nova instância para manter o mínimo de `2`.
* **Administração segura:** Conectando à instância pelo *Session Manager* sem uso de chave SSH exposta.
* **Escalonamento por carga:** Simulando aumento de uso de CPU para observar o comportamento do alarme e do ASG.

---

## 💡 Principais Insights

* **Padronização via Launch Template:** Evita configurações manuais repetitivas e permite que novas instâncias subam de forma previsível.
* **Acesso sem porta 22:** O *Session Manager* reduz a exposição de rede e centraliza o acesso administrativo com base em permissões IAM.
* **Limpeza organizada:** Encerrar recursos na ordem correta evita instâncias sendo recriadas pelo ASG ou cobranças residuais.

---

## 📁 Documentação Complementar no Repositório

* [`docs/anotacoes-praticas.md`](docs/anotacoes-praticas.md) — Anotações detalhadas da experiência prática.
* [`docs/checklist-finops.md`](docs/checklist-finops.md) — Passos para desligar/remover os recursos ao fim do lab.
* [`docs/referencias.md`](docs/referencias.md) — Documentações oficiais e guias de apoio.
* [`images/arquitetura-drawio.png`](images/arquitetura-drawio.png) — Diagrama da arquitetura.