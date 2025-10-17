# RELATÓRIO DE IMPLEMENTAÇÃO DE SERVIÇOS AWS

**Data:** 17/10/2025  
**Empresa:** Abstergo Industries 
**Responsável:** Samara da Trindade  

---

## Introdução  

A **Abstergo Industries** é uma farmácia digital que iniciou suas operações com um site hospedado em um **servidor local (on-premises)**.  
Com o crescimento rápido da base de clientes e aumento nas vendas online, o servidor começou a apresentar **lentidão, indisponibilidade em horários de pico** e **altos custos de manutenção física**.  

Para resolver esses problemas e garantir uma operação estável, segura e escalável, foi planejada a **migração gradual da infraestrutura para a AWS (Amazon Web Services)**.  

O objetivo deste projeto foi **demonstrar a diferença prática entre um ambiente on-premises e um ambiente em nuvem**, simulando a modernização da farmácia digital com o uso de **três serviços essenciais da AWS**:  
- **Amazon EC2** (computação elástica),  
- **Amazon S3** (armazenamento em nuvem),  
- **Amazon CloudWatch** (monitoramento e métricas).  

---

## Cenário Atual (On-Premises)  

Antes da migração, o ambiente da Abstergo funcionava da seguinte forma:

- Um **servidor físico** localizado na própria farmácia, responsável por hospedar o site e o banco de dados.  
- O servidor tinha **limitação de hardware** (16 GB RAM, 1 TB HDD, CPU limitada).  
- Em horários de pico como campanhas de desconto ou lançamento de produtos, o site **ficava fora do ar por sobrecarga**.  
- Era necessário **comprar novos servidores** e **interromper operações** sempre que o tráfego aumentava.  
- Os **custos de energia, refrigeração e manutenção técnica** eram altos e inflexíveis.  

### Principais Desafios:
- Falta de **escalabilidade automática**;  
- Risco de **falhas físicas** e perda de dados;  
- Ausência de **monitoramento proativo**;  
- **Tempo de resposta lento** para os clientes em horários críticos.

---

## Solução Proposta: Migração para AWS  

A estratégia adotada foi **replicar o ambiente on-premises na nuvem**, aproveitando os recursos sob demanda e o modelo de pagamento baseado em uso da AWS.  
Com isso, a empresa passou a pagar **apenas pelo que consome**, sem necessidade de investimento em novos equipamentos.  

As etapas da implementação foram divididas conforme os principais serviços utilizados:

---

### Etapa 1 – `Amazon S3 (Simple Storage Service)`  

- **Objetivo:** substituir o armazenamento físico por armazenamento escalável e seguro em nuvem.  
- **Descrição:**  
  Criou-se um bucket chamado `abstergo-website-assets` para hospedar imagens, documentos e relatórios do sistema da farmácia.  
  Com o versionamento habilitado, os arquivos têm **backup automático** e podem ser **recuperados em caso de exclusão acidental**.  
- **Benefício:**  
  Redução de custos com HDs locais e eliminação de riscos de perda de dados por falha física.  
- **Aprendizado técnico:**  
  Configuração de políticas IAM, compreensão de custos por requisição e boas práticas de segurança com **bloqueio de acesso público**.

---

### Etapa 2 – `Amazon EC2 (Elastic Compute Cloud)`  

- **Objetivo:** hospedar o site da farmácia em uma instância EC2 com alta disponibilidade.  
- **Descrição:**  
  Foi criada uma instância, configurada para rodar um servidor web Apache com o site da Abstergo.  
  Utilizou-se o **Elastic IP** para manter um endereço fixo e **Security Groups** para permitir tráfego HTTP (porta 80) e SSH (porta 22).  
- **Benefício:**  
  Quando o número de requisições aumentou, foi possível **escalar horizontalmente** (Auto Scaling) e **distribuir a carga com um Load Balancer**, sem precisar comprar novos servidores físicos.  
- **Aprendizado técnico:**  
  Entendimento de **tipos de instâncias**, **modelos de cobrança (on-demand vs reserved)** e **boas práticas de provisionamento seguro**.

---

### Etapa 3 – `Amazon CloudWatch`  

- **Objetivo:** monitorar os serviços da AWS e gerar métricas em tempo real.  
- **Descrição:**  
  O CloudWatch foi configurado para monitorar o uso de CPU, memória e disco da instância EC2, além de acompanhar o número de acessos ao bucket S3.  
  Foram criados **dashboards visuais** e **alarmes automáticos** para alertar sobre picos de tráfego e uso de recursos.  
- **Benefício:**  
  A empresa passou a ter **visibilidade total do desempenho da aplicação** e a **tomar decisões baseadas em dados**, reduzindo incidentes e custos desnecessários.  
- **Aprendizado técnico:**  
  Configuração de **alarmes automáticos**, **logs centralizados** e análise de métricas para otimização de performance.

---

## Resultados da Migração  

| Categoria | Ambiente On-Premises | Ambiente AWS |
|------------|----------------------|---------------|
| **Escalabilidade** | Limitada a um único servidor físico | Escalabilidade automática com EC2 e Load Balancer |
| **Custo** | Alto investimento inicial em hardware | Pagamento sob demanda (pay-as-you-go) |
| **Disponibilidade** | Risco de falhas locais e downtime | Alta disponibilidade (99,99%) |
| **Segurança** | Controle físico restrito | IAM, Security Groups e criptografia integrada |
| **Monitoramento** | Inexistente | CloudWatch com métricas em tempo real |
| **Backup e Recuperação** | Manual, sujeito a erro | Automático via S3 e versionamento |

---

## Conclusão  

A migração da infraestrutura on-premises para a **AWS** proporcionou à *Abstergo Industries* uma operação mais **flexível, segura e escalável**, alinhada às demandas de crescimento do site da farmácia digital.  

Além da redução de custos e aumento de desempenho, o projeto serviu como **laboratório prático de aprendizado em computação em nuvem**, demonstrando como pequenas e médias empresas podem se beneficiar da adoção de serviços AWS.  

Como próximos passos, recomenda-se:  
- Implementar o **Amazon RDS** para o banco de dados da aplicação;  
- Testar **AWS Lambda** para funções automatizadas;  
- Configurar um **CloudFront** para distribuição global do conteúdo da farmácia.  

---

## Anexos  
- Diagramas de arquitetura (AWS EC2 + S3 + CloudWatch).  
- Prints de configuração e métricas.  
- Planilha de custos simulada no **AWS Pricing Calculator**.  

---

**Assinatura do Responsável pelo Projeto:**  
Samara da Trindade
