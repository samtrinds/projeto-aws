
---

```markdown
# Implementando o projeto da Abstergo Industries (Free Tier)

> **Contexto:**  
> Projeto pessoal e fictício criado para estudar e testar conhecimentos de AWS.  
> Todo o ambiente foi configurado usando o **Free Tier da AWS (2025)**, com foco em praticar a criação de uma infraestrutura web escalável e monitorada.

---

## Introdução e Estrutura Geral

Este guia foi criado para mostrar na prática como seria a criação do ambiente completo na AWS, usando apenas os recursos gratuitos do Free Tier.  
A ideia é aprender na prática como montar uma estrutura de nuvem real, desde o controle de acesso até a automação e monitoramento.

A arquitetura final será composta por:
- Uma conta segura e configurada com usuário IAM e autenticação MFA  
- Um bucket S3 para armazenar arquivos e imagens  
- Uma instância EC2 com Apache para hospedar um site simples  
- Um Application Load Balancer (ALB) distribuindo o tráfego  
- Um Auto Scaling Group (ASG) para manter a disponibilidade  
- Monitoramento via CloudWatch com métricas e alarmes

---

### Etapas do projeto

- Preparar conta e IAM (usuário e MFA)  
- Criar bucket S3 para assets com versionamento e bloqueio de acesso público  
- Lançar uma instância EC2 (Amazon Linux) com script de inicialização e CloudWatch Agent  
- Criar Target Group e Application Load Balancer (ALB)  
- Criar Launch Template e Auto Scaling Group (ASG) integrados ao ALB  
- Configurar CloudWatch com dashboards e alarmes  
- Fazer testes de carga e acompanhar o consumo dentro do Free Tier  
- Encerrar os recursos ao final do projeto para evitar cobranças

---

## 1) Criar um grupo IAM e um usuário de deploy

Antes de começar a criar os recursos, é importante garantir que sua conta esteja configurada com boas práticas de segurança.  
Nunca use a conta root para tarefas do dia a dia. O ideal é criar um usuário com permissões específicas e habilitar autenticação MFA (Multi-Factor Authentication).

### Passos

1. No console da AWS, procure por **IAM** e acesse o serviço.  
2. No menu lateral, clique em **User groups** e depois em **Create group**.  
3. Nomeie o grupo como `adm-abstergo`.  
4. Em **Attach permissions policies**, **evite** anexar múltiplas policies `*FullAccess` sem critério. Para fins de estudo você pode:
   - Anexar a policy gerenciada `AdministratorAccess` para um grupo administrativo *temporariamente* enquanto aprende, **ou**
   - (melhor) **criar uma policy personalizada** que contenha apenas as permissões necessárias (EC2, S3, CloudWatch, ElasticLoadBalancing, AutoScaling) seguindo o princípio do menor privilégio.
5. Clique em **Create group**.

6. Agora vá até **Users** e clique em **Create user**.  
7. Nomeie o usuário (por exemplo, `abstergo-admin`).  
8. Marque a opção para permitir acesso ao console da AWS.  
9. Escolha gerar uma senha automática (ou defina uma manual).  
10. Marque a opção para o usuário trocar a senha no primeiro login.  
11. Clique em **Next** e adicione o usuário ao grupo `adm-abstergo`.  
12. Clique em **Create user**.

13. Depois que o usuário for criado, abra o perfil dele e vá até a aba **Security credentials**.  
14. Configure o MFA virtual com um aplicativo como Google Authenticator ou Authy.  
15. Anote o **Login URL personalizado** (IAM sign-in URL). Ele será usado para fazer login com esse usuário em vez da conta root.

## 2) Criar bucket S3 para assets

O Amazon S3 será usado para armazenar arquivos estáticos, como imagens, scripts e documentos do site. Mesmo em projetos simples, é importante configurar o bucket corretamente para garantir segurança e versionamento.

### Passos

1. No console da AWS, busque por **S3** e clique em **Create bucket**.  
2. Escolha um nome único, por exemplo: `abstergo-assets`.  
3. Escolha a região mais próxima de você (ex: `sa-east-1`).  
4. **Mantenha** o bloqueio de acesso público ativado por padrão (Block all public access) a menos que você tenha um motivo explícito para tornar objetos públicos.  
5. Em **Bucket Versioning**, ative **Enable versioning**. Isso garante que versões antigas de arquivos possam ser recuperadas caso algo seja sobrescrito.  
6. Mantenha as outras opções padrão e clique em **Create bucket**.

### Testando o bucket

1. Abra o bucket criado.  
2. Vá até a aba **Properties** e confirme que o **versioning** está ativo.  
3. Teste o bloqueio de acesso público:  
   - Copie o **Object URL** e tente abrir no navegador.  
   - O acesso deve ser negado (403 Forbidden), o que é o comportamento esperado.

## 3) Lançar uma instância EC2 com webserver e CloudWatch Agent

Será criada uma instância EC2 que servirá como servidor web principal do projeto, a configuração inicial incluirá a instalação automática de um servidor HTTP (Apache) e do agente do CloudWatch para monitoramento.

### Passos

1. No console da AWS, procure por **EC2** e clique em **Launch instance**.  
2. Nomeie a instância como `abstergo-web-01`.  
3. Em **Application and OS Images (AMI)**, selecione **Amazon Linux 2023 (Free Tier eligible)** ou a AMI compatível indicada pelo console.  
4. Em **Instance type**, escolha `t2.micro` (gratuito no Free Tier),é interessante confirmar que a AMI escolhida suporta o tipo `t2.micro`.  
5. Crie um novo par de chaves (`abstergo-keypair`) e baixe o arquivo `.pem`.  
6. Em **Network settings**, crie um novo **Security Group** com as seguintes regras:  
   - **SSH (22)**: acesso permitido apenas do seu IP  
   - **HTTP (80)**: acesso permitido de qualquer origem (0.0.0.0/0)  
7. Em **User data**, cole o seguinte script para instalar o servidor web e o CloudWatch Agent automaticamente:

---

```bash
#!/bin/bash
yum update -y
yum install -y httpd amazon-cloudwatch-agent
systemctl enable httpd
systemctl start httpd

echo "<h1>Abstergo Web Server - $(hostname)</h1>" > /var/www/html/index.html

# Configuração básica do CloudWatch Agent
cat <<EOF > /opt/aws/amazon-cloudwatch-agent/bin/config.json
{
  "metrics": {
    "append_dimensions": {
      "InstanceId": "\${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": ["disk_used_percent"],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      }
    }
  }
}
EOF

/opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s


```

8. Clique em executar instance 
9.  Após alguns minutos, acesse a aba Instances e copie o Public IPv4 address.
10. Digite http://IP - Deve aparecer a página simples com o título Abstergo Web Server.

## Verificação do CloudWatch Agent

Vá até o serviço CloudWatch.

Acesse Metrics → CWAgent e verifique se há métricas de memória e disco sendo registradas.

Se as métricas estiverem chegando, significa que o agente está ativo e enviando dados corretamente.

Agora sua instância EC2 está configurada, com o Apache rodando e o CloudWatch Agent monitorando o desempenho.


## 4) Criar Target Group e Application Load Balancer (ALB)

Agora que a instância EC2 está pronta e servindo o site, o próximo passo é criar um Load Balancer para distribuir o tráfego entre múltiplas instâncias. Mesmo que neste momento temos apenas uma instância, o ALB já será configurado para suportar escalabilidade automática.

### Criar o Target Group

1. No console da AWS, vá até **EC2 → Balanceamento de carga → Target Groups**.  
2. Clique em **Create target group**.  
3. Escolha o tipo **Instances**.  
4. Nomeie como `abstergo-target`.  
5. Em **Protocol**, selecione **HTTP** e a porta **80**.  
6. Escolha a mesma VPC usada pela sua instância EC2.  
7. Mantenha as configurações padrão de **Health checks** (HTTP → `/`).  
8. Clique em **Next**.  
9. Na lista de instâncias, selecione sua instância `abstergo-web-01` e clique em **Include as pending below**.  
10. Finalize clicando em **Create target group**.

### Criar o Load Balancer

1. No console da AWS, acesse **EC2 → Load Balancers → Create Load Balancer**.  
2. Escolha **Application Load Balancer (ALB)**.  
3. Nomeie como `abstergo-alb`.  
4. Selecione **Internet-facing** e **IPv4**.  
5. Escolha a mesma VPC e selecione pelo menos duas subnets em zonas diferentes (para alta disponibilidade).  
6. Em **Security Groups**, selecione o mesmo grupo de segurança que permite tráfego HTTP (porta 80).  
7. Em **Listeners and routing**, adicione um listener HTTP (porta 80) e selecione o target group `abstergo-target`.  
8. Clique em **Create load balancer**.

### Testando o ALB

1. Após alguns minutos, vá até o ALB recém-criado e copie o **DNS name** exibido.  
2. Cole o endereço no navegador.  
3. A página do servidor web da Abstergo deve aparecer normalmente.  
4. Para confirmar que o balanceamento está ativo, adicione uma segunda instância e verifique se ambas aparecem como **healthy** no Target Group.

O ALB agora está configurado e funcionando, pronto para ser integrado ao Auto Scaling Group.

## 5) Criar Launch Template e Auto Scaling Group (ASG)

Agora vamos automatizar a criação e substituição de instâncias EC2.  
O Launch Template define a configuração base da instância, e o Auto Scaling Group usa esse modelo para criar novas instâncias automaticamente, mantendo o ambiente sempre disponível.

### Criar Launch Template

1. No console da AWS, vá até **EC2 → role até a seção **Instances** → *Launch Templates** (fica logo abaixo de “Instances Types”).  
2. Clique em **Create launch template**.  
3. Em **Launch template name**, digite `abstergo-template`.  
4. Em **Version description**, escreva `versão inicial`.  
5. Escolha a AMI compatível selecionada anteriormente.  
6. Em **Instance type**, selecione `t2.micro`.  
7. Em **Key pair**, use a chave que você criou antes.  
8. Em **Network settings**, deixar sem incluir a VPC/subnet no template é aceitável (o ASG irá escolher as subnets ao criar as instâncias).  
9. Em **Security groups**, selecione o grupo que permite tráfego HTTP (porta 80).  
10. Mantenha o volume padrão de 8 GiB.  
11. Em **Advanced details**, cole o seguinte script em User data:

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl enable httpd
systemctl start httpd
echo "<h1>Bem-vindo à Abstergo Industries - Servidor Auto Scaling</h1>" > /var/www/html/index.html

```
12. Clique em Create launch template.
13. Verifique se ele aparece com status Available.

## Criar Auto Scaling Group (ASG)

1. Ainda dentro do serviço EC2, vá até **Auto Scaling → Auto Scaling Groups**.  
2. Clique em **Create Auto Scaling group**.  
3. Em **Auto Scaling group name**, digite `abstergo-asg`.  
4. Selecione o launch template criado (`abstergo-template`).  
5. Clique em **Next**.  
6. Escolha a mesma **VPC** usada anteriormente.  
7. Selecione **duas subnets públicas** (de zonas de disponibilidade diferentes).  
8. Em **Load balancing**, selecione **Attach to an existing load balancer**.  
9. Escolha o **target group abstergo-target**.  
10. Marque **Enable ELB health checks**.  
11. Clique em **Next**.

---

## Configurar tamanho e políticas

1. Em **Group size**, defina:  
   - Desired capacity: 1  
   - Minimum capacity: 1  
   - Maximum capacity: 2  
2. Em **Scaling policies**, selecione **Target tracking scaling policy**.  
3. Escolha a métrica **Average CPU utilization**.  
4. Defina o valor-alvo como **50%**.  
5. Clique em **Next** e depois em **Skip adding notifications**.  
6. Adicione uma tag:  
   - Key: Name  
   - Value: abstergo-asg-instance  
7. Clique em **Create Auto Scaling group**.  
8. Aguarde 2 a 3 minutos para o grupo criar a instância inicial.

---

## Verificar funcionamento

1. Vá até **EC2 → Instances**.  
2. Você verá uma nova instância criada automaticamente pelo ASG.  
3. Quando estiver com status **running**, verifique o **Security Group** e o **User data**.  
4. Vá até o painel do **Load Balancer** e copie o **DNS público**.  
5. Cole o endereço no navegador — deve aparecer a mensagem:  
   **Bem-vindo à Abstergo Industries - Servidor Auto Scaling**.  
6. Se quiser testar, pare a instância manualmente.  
   O ASG criará outra automaticamente em poucos minutos para manter o ambiente ativo.

# Você também pode criar o Launch Template e o Auto Scaling Group diretamente pelo CloudShell, usando os comandos abaixo:

aws ec2 create-launch-template
--launch-template-name abstergo-template
--version-description "v1"
--launch-template-data '{
"ImageId":"ami-0c55b159cbfafe1f0",
"InstanceType":"t2.micro",
"SecurityGroupIds":["sg-xxxxxxxx"],
"UserData":"IyEvYmluL2Jhc2gKeXVtIHVwZGF0ZSAtCnRvdWNoIC9ldGMvaHR0cGQvaHR0cGQub24KCnl1bSBpbnN0YWxsIC15IGh0dHBkCnN5c3RlbWN0bCBlbmFibGUgaHR0cGQKc3lzdGVtY3RsIHN0YXJ0IGh0dHBkCmVjaG8gIjxoMT5CZW0tdmluZG8gw6AgQWJzdGVyZ28gSW5kdXN0cmllcyAtIFNlcnZpZG9yIEF1dG8gU2NhbGluZzwvaDE+IiA+IC92YXIvd3d3L2h0bWwvaW5kZXguaHRtbA=="
}'

aws autoscaling create-auto-scaling-group
--auto-scaling-group-name abstergo-asg
--launch-template LaunchTemplateName=abstergo-template
--min-size 1
--max-size 2
--desired-capacity 1
--vpc-zone-identifier "subnet-xxxxxx,subnet-yyyyyy"
--target-group-arns "arn:aws:elasticloadbalancing:..."

## Monitoramento e CloudWatch

Nós automatizamos isso com o script no passo de lançar uma instância, mas, caso queira saber como faz isso pelo console, aqui está:

1. Vá até o serviço **CloudWatch** no console da AWS.  
2. No menu lateral, acesse **Dashboards** e clique em **Create dashboard**.  
3. Nomeie o dashboard como `abstergo-monitor`.  
4. Adicione um widget do tipo **Line**.  
5. Escolha a métrica **EC2 → Per-Instance Metrics → CPUUtilization**.  
6. Selecione a instância ou o Auto Scaling Group ativo.  
7. Clique em **Add to dashboard**.  
8. Adicione também widgets para:  
   - NetworkIn  
   - NetworkOut  
   - StatusCheckFailed  
   - DiskUsedPercent (se configurado via CloudWatch Agent)  
9. Ajuste os gráficos conforme desejar e clique em **Save dashboard**.

---

## Criar Alarmes

1. No menu lateral, acesse **Alarms → All alarms**.  
2. Clique em **Create alarm**.  
3. Escolha a métrica **EC2 → Per-Instance Metrics → CPUUtilization**.  
4. Clique em **Select metric**.  
5. Defina a condição:  
   - Quando **CPUUtilization ≥ 70%** por **5 minutos**.  
6. Clique em **Next**.  
7. Em **Notification**, selecione **Create new topic**.  
   - Nome do tópico: `abstergo-alerts`  
   - Informe um e-mail para receber os alertas.  
8. Confirme o e-mail recebido para ativar o SNS.  
9. Clique em **Next** e depois em **Create alarm**.

---

## Verificação de Métricas

Se você adicionou o trecho de configuração do **CloudWatch Agent** no script de inicialização (User Data), o envio de métricas de memória e disco já estará ativo automaticamente.  

1. No console da AWS, vá até **CloudWatch → Metrics → CWAgent**.  
2. Verifique se as métricas de memória (`mem_used_percent`) e disco (`disk_used_percent`) estão sendo registradas.  
3. Caso apareçam, o agente está funcionando corretamente e o monitoramento está ativo.

## 6) Limpeza de Recursos e Encerramento do Projeto

Quando terminar os testes e não precisar mais do ambiente, é importante remover todos os recursos para evitar custos fora do Free Tier.  
Siga a ordem abaixo para garantir que tudo seja encerrado corretamente.

### Encerrar instâncias EC2

1. Vá até **EC2 → Instances**.  
2. Selecione as instâncias criadas manualmente e as gerenciadas pelo Auto Scaling Group.  
3. Clique em **Instance state → Terminate instance**.  
4. Confirme a exclusão.

---

### Excluir o Auto Scaling Group

1. No console da AWS, vá até **EC2 → Auto Scaling Groups**.  
2. Selecione o grupo `abstergo-asg`.  
3. Clique em **Delete**.  
4. Confirme a exclusão.

---

### Excluir o Launch Template

1. Vá até **EC2 → Launch Templates**.  
2. Selecione `abstergo-template`.  
3. Clique em **Actions → Delete template**.  
4. Confirme.

---

### Excluir o Load Balancer e Target Group

1. Vá até **EC2 → Load Balancers**.  
2. Selecione `abstergo-alb` e clique em **Actions → Delete**.  
3. Vá até **Target Groups**, selecione `abstergo-target` e clique em **Delete**.  
4. Confirme ambos.

---

### Excluir o bucket S3

1. Vá até o serviço **S3**.  
2. Selecione o bucket criado (ex: `abstergo-assets`).  
3. Clique em **Empty** para esvaziar o conteúdo.  
4. Depois clique em **Delete bucket** e confirme.

---

### Verificar outros serviços

1. Vá até **CloudWatch → Dashboards** e exclua o dashboard `abstergo-monitor`.  
2. Em **Alarms**, exclua os alarmes criados para CPU.  
3. Em **SNS**, remova o tópico `abstergo-alerts` se quiser.

---

### Confirmar na aba Billing

1. Vá até o console **Billing and Cost Management**.  
2. Verifique se não há recursos ativos sendo cobrados.  
3. Confirme que o consumo está dentro do limite **Free Tier**.

---

### Conclusão

Com isso, o ambiente AWS foi completamente configurado, testado e encerrado dentro do Free Tier.  

- EC2 com servidor web ativo  
- S3 para armazenamento  
- ALB para balanceamento de carga  
- Auto Scaling configurado  
- Monitoramento com CloudWatch  


Assinatura: Samara da Trindade 

