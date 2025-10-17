## Implementando o projeto

Lembrando que é um projeto pessoal e ficticio para testar conhecimento, então o ambiente está sendo todo produzido no free tier. 
1. Preparar conta e IAM (user + MFA)

2. Criar bucket S3 (assets) com versionamento + bloquear acesso público

3. Lançar uma EC2 (Amazon Linux ) com user-data para instalar webserver + CloudWatch Agent

4. Criar Target Group e Application Load Balancer (ALB)

5. Criar Launch Template/Configuration + Auto Scaling Group (ASG) ligado ao ALB

6. Configurar CloudWatch: dashboards & alarmes; instalar CloudWatch Agent para métricas de memória/disk

7. Testar com carga leve; monitorar billing/Free Tier 

8. Limpar recursos quando terminar (para evitar cobrança rsrs)

