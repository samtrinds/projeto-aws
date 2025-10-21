# 🧩 RELATÓRIO DE IMPLEMENTAÇÃO DE SERVIÇOS AWS  

**Data:** 17/10/2025  
**Empresa:** Abstergo Industries  
**Responsável:** Samara da Trindade  

---

## Introdução  

A **Abstergo Industries**, uma farmácia digital em expansão, enfrentava desafios de desempenho e disponibilidade ao hospedar seu site em um **servidor físico local (on-premises)**.  
Com o aumento do número de acessos, o servidor passou a sofrer com **lentidão, instabilidade** e **custos crescentes de manutenção**.

Para solucionar esses problemas e aprimorar a escalabilidade do ambiente, foi planejada a **migração gradual da infraestrutura para a AWS (Amazon Web Services)**, aproveitando os recursos disponíveis no **Free Tier** para fins de estudo e validação prática.  

O projeto teve como principal objetivo **demonstrar na prática a modernização de um ambiente local para a nuvem**, utilizando três serviços principais da AWS:

- **Amazon EC2** - Hospedagem e processamento de aplicações  
- **Amazon S3** - Armazenamento seguro e escalável  
- **Amazon CloudWatch** - Monitoramento e métricas em tempo real  

---

## Cenário Atual (On-Premises)  

Antes da migração, o ambiente da Abstergo funcionava com um **único servidor físico** hospedando o site e o banco de dados da farmácia.  
Esse modelo apresentava diversas limitações:

- Capacidade de hardware restrita (16 GB RAM, 1 TB HDD, CPU limitada)  
- Falhas frequentes em horários de pico  
- Ausência de redundância e backups automatizados  
- Necessidade de intervenções manuais para manutenção e escalonamento  

### Principais Desafios

- Falta de **escalabilidade automática**  
- **Risco de falhas físicas** e perda de dados  
- Inexistência de **monitoramento proativo**  
- **Tempo de resposta lento** durante promoções e campanhas  

---

## Solução Proposta: Migração para a AWS  

A migração teve como meta **replicar e modernizar o ambiente on-premises** dentro da infraestrutura da AWS, garantindo elasticidade, segurança e economia.  
Foram utilizados serviços nativos e recursos gratuitos do **AWS Free Tier (2025)**, estruturando o ambiente com base em boas práticas de nuvem.

A implementação foi dividida nas seguintes etapas:

---

## 1) - Armazenamento com Amazon S3  

**Objetivo:** substituir o armazenamento físico local por um ambiente escalável e seguro.  

- Criado o bucket **`abstergo-website-assets`**, configurado com **versionamento habilitado** e **bloqueio de acesso público**.  
- O S3 passou a armazenar arquivos estáticos do site (imagens, relatórios e documentos).  
- O versionamento garante **backup automático e recuperação de versões antigas**.  

**Benefícios:**  
- Eliminação de riscos de perda física de dados  
- Custos sob demanda, sem necessidade de expansão de hardware  
- Facilidade de integração com outros serviços AWS  

**Aprendizado técnico:**  
- Políticas de acesso IAM  
- Controle de acesso via ACLs e políticas de bucket  
- Entendimento do modelo de cobrança por requisição  

---

## 2) Computação com Amazon EC2  

**Objetivo:** hospedar o site da Abstergo de forma escalável e disponível.  

- Lançada uma instância EC2 (Amazon Linux) configurada via **User Data** para instalar automaticamente o servidor web Apache e o **CloudWatch Agent**.  
- Criado um **Launch Template** (`abstergo-template`) para padronizar novas instâncias.  
- Implementado um **Auto Scaling Group (ASG)** conectado a um **Application Load Balancer (ALB)**, distribuindo o tráfego entre múltiplas instâncias.  
- O **Security Group** foi configurado para permitir apenas tráfego HTTP (porta 80) e SSH (porta 22).  

**Benefícios:**  
- Alta disponibilidade e escalabilidade automática  
- Balanceamento de carga inteligente via ALB  
- Substituição automática de instâncias em caso de falhas  

**Aprendizado técnico:**  
- Entendimento de tipos de instância (t2.micro para free tier)  
- Criação e vinculação de ASG, Target Groups e Load Balancer  
- Automação via User Data e políticas de escalonamento  

---

## 3) Monitoramento com Amazon CloudWatch  

**Objetivo:** obter visibilidade total da performance da infraestrutura.  

- O **CloudWatch Agent** foi configurado para coletar métricas de **CPU**, **memória** e **uso de disco** diretamente das instâncias EC2.  
- Criado um **dashboard** personalizado (`abstergo-monitor`) com gráficos de:
  - CPUUtilization  
  - NetworkIn / NetworkOut  
  - DiskUsedPercent  
  - MemUsedPercent  
- Configurados **alarmes automáticos** via **Amazon SNS**, enviando alertas por e-mail quando o uso de CPU ultrapassava 70%.  

**Benefícios:**  
- Monitoramento em tempo real e alertas automáticos  
- Identificação rápida de gargalos e consumo de recursos  
- Base de dados para decisões preventivas e ajustes de escala  

**Aprendizado técnico:**  
- Configuração do CloudWatch Agent via JSON  
- Criação de métricas personalizadas e dashboards  
- Integração com SNS para notificações automatizadas  

---

## Resultados Obtidos  

| Critério | Ambiente On-Premises | Ambiente AWS |
|-----------|----------------------|---------------|
| **Escalabilidade** | Limitada a um único servidor físico | Escalabilidade automática com EC2 e ASG |
| **Custo** | Alto investimento em hardware | Pagamento sob demanda, dentro do Free Tier |
| **Disponibilidade** | Sujeita a falhas locais | Alta disponibilidade com ALB e múltiplas Zonas de Disponibilidade |
| **Segurança** | Controle físico restrito | IAM, Security Groups e criptografia integrada |
| **Monitoramento** | Inexistente | CloudWatch com métricas e alarmes em tempo real |
| **Backup e Recuperação** | Manual | Automático via S3 e versionamento |

---

## Encerramento e Limpeza de Recursos  

Após os testes e validações, todos os recursos foram devidamente encerrados para evitar custos fora do Free Tier.  
Foram removidos:

- Instâncias EC2 e Auto Scaling Group  
- Launch Template e Load Balancer  
- Bucket S3 `abstergo-website-assets`  
- Dashboards e alarmes do CloudWatch  
- Tópico SNS de alertas (`abstergo-alerts`)  

O encerramento garantiu que nenhum recurso ativo permanecesse gerando cobrança indevida.

---

## Conclusão  

A migração experimental da **Abstergo Industries** para a AWS demonstrou como um ambiente tradicional pode ser **modernizado, escalável e seguro** utilizando serviços essenciais da nuvem.  

Além de reduzir custos operacionais e eliminar limitações físicas, o projeto proporcionou **aprendizado prático** em automação, monitoramento e boas práticas de arquitetura AWS.  

### Próximos Passos

- Implementar o **Amazon RDS** para o banco de dados da aplicação  
- Testar **AWS Lambda** para automação de tarefas  
- Adicionar **Amazon CloudFront** para distribuição global do conteúdo  

---

## Anexos  

Consulte o **Implementação AWS Free Tier (Abstergo Industries)** para detalhes sobre a configuração passo a passo do ambiente.  

---

**Assinatura do Responsável pelo Projeto:**  
Samara da Trindade  
