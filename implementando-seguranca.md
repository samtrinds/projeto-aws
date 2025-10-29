# Implementando o projeto de segurança (Abstergo Industries)

## Introdução 

Neste documento serão demonstradas as etapas práticas para habilitar e configurar os três serviços de segurança dentro do ambiente já existente. Cada parte foi pensada para funcionar dentro do Free Tier, permitindo o uso gratuito para fins de estudo e validação.

A ordem de configuração será:

- Ativar o AWS KMS e criar uma chave de criptografia gerenciada
- Configurar o AWS WAF integrado ao Application Load Balancer
- Habilitar o AWS Security Hub e verificar as recomendações de segurança

## Etapas

1) **Criar e configurar uma chave no AWS Key Management Service (KMS)**

- No console da AWS, procure por Key Management Service (KMS).
- Clique em Customer managed keys e depois em Create key.
- Em Key type, escolha Symmetric.
- Dê um nome à chave, por exemplo abstergo-kms-key.
- Em Administrators, selecione um grupo ou role administrativo (por exemplo um role `kms-admin`) em vez de atribuir permissões diretamente a um usuário individual. Isso facilita auditoria, rotação de pessoal e segue o princípio do menor privilégio.
- Em Key users, adicione apenas as funções/serviços que realmente precisarão acessar essa chave (por exemplo, a role associada às instâncias EC2 ou a role usada pelo serviço que lerá segredos).
- Ative **Automatic key rotation** (rotação automática anual) para chaves simétricas para melhorar a segurança.
- Configure a policy da chave (key policy) garantindo que apenas administradores autorizados possam alterar configurações e que os serviços autorizados possam usá-la.
- Clique em Next e depois em Create key.

A chave criada servirá para criptografar dados diretamente em serviços como o S3, RDS, EBS e também em variáveis de ambiente gerenciadas.

## Protegendo variáveis sensíveis
- No console, abra o serviço AWS Secrets Manager.
- Clique em Store a new secret.
- Escolha Other type of secrets.
- Insira as credenciais que normalmente ficariam no arquivo .env, como: DB_USER DB_PASSWORD API_KEY
- Em Encryption key, selecione a chave criada no KMS (abstergo-kms-key).
- Dê um nome ao segredo, por exemplo abstergo-env-secrets.
- Em Resource policy/Access, garanta que apenas as roles/serviços que precisam ler o segredo tenham a permissão `secretsmanager:GetSecretValue`.
- Clique em Next até concluir.

Agora, suas variáveis estão criptografadas. Para usá-las na aplicação, basta recuperar o segredo de forma segura via SDK ou CLI, usando roles (assume role) quando possível, sem deixá-las expostas no código.

2) **Configurar o AWS Web Application Firewall (WAF)**

O WAF protegerá o site hospedado na instância EC2 por meio do Application Load Balancer (ALB), bloqueando ataques de aplicação.

- No console da AWS, procure por WAF e abra o serviço.
- Clique em Create web ACL.
- Dê um nome, por exemplo abstergo-waf.
- Em Resource type, selecione Application Load Balancer.
- Escolha o ALB que foi configurado no projeto anterior (abstergo-alb).
- Clique em Next para adicionar as regras.

Adicione regras gerenciadas:
- Selecione Add managed rule groups.
- Escolha AWS Managed Rules.

Adicione os seguintes grupos:
- AWSManagedRulesCommonRuleSet (protege contra SQL Injection e XSS)
- AWSManagedRulesAmazonIpReputationList (bloqueia IPs maliciosos conhecidos)
- AWSManagedRulesKnownBadInputsRuleSet (impede entrada de payloads suspeitos)
- Clique em Add rules.

Observação importante sobre ações e modo de teste:
- Cada regra gerenciada tem sua **action** (Block, Count, Allow). **Durante a fase de teste inicial, configure as regras em `Count`** para observar o que seria bloqueado sem impactar usuários reais.  
- **O campo Default action do Web ACL só aceita `Allow` ou `Block`**. Para boa prática, após avaliar o tráfego com as regras em `Count` e ajustar falsos positivos, altere as regras para `Block` e **defina Default action = Block** (negar por padrão), criando explicitamente exceções/allow rules quando necessário.  
- Não deixe o Default action em `Allow` em produção sem antes ter validado as regras em modo `Count`. Allow como default pode deixar você exposto enquanto regras não estiverem corretamente afinadas.

- Clique em Next, revise e finalize com Create web ACL.

**O WAF começará a analisar todas as requisições que chegam ao seu site e, após validação, bloqueará automaticamente as ameaças detectadas quando as regras estiverem em modo Block.**

- Testando o WAF
- Acesse o serviço WAF → Web ACLs.
- Selecione abstergo-waf.
- Clique em Monitoring e observe as métricas de requisições bloqueadas e em modo Count.
- Realize testes simples de requisições suspeitas, como parâmetros incomuns na URL, e verifique se foram detectados (em `Count`) antes de alternar para `Block`.

3) **Habilitar e configurar o AWS Security Hub**

O Security Hub será usado para centralizar a segurança da conta e avaliar as configurações com base nas boas práticas da AWS.

- No console, procure por Security Hub.
- Clique em Go to Security Hub.
- Selecione Enable Security Hub.
- Escolha a região onde seus recursos estão hospedados.
- Após ativar, o serviço exibirá um painel com a opção Security Standards.
- Ative o padrão AWS Foundational Security Best Practices (verifique a versão disponível no console e ative a versão mais atual).

**O Security Hub iniciará uma análise da conta e mostrará recomendações automáticas, como:**

- Usuários sem MFA habilitado
- Buckets S3 públicos
- Políticas IAM excessivamente permissivas
- Recursos sem criptografia habilitada
- Criar alertas integrados com CloudWatch
- No painel do Security Hub, acesse Insights.
- Escolha uma verificação importante, como MFA Disabled for Root Account.
- Clique em Create CloudWatch rule.
- Configure um alerta para ser enviado via SNS.
- Adicione seu e-mail e confirme o recebimento da notificação.

## Conclusão

Após os testes, é importante revisar o consumo para manter o uso dentro do Free Tier. Nenhum dos três serviços gera custos elevados se configurados com cuidado, mas é sempre bom monitorar. Verifique no painel de Billing and Cost Management se não há cobranças inesperadas, quando o estudo for concluído, é possível desativar os serviços sem perder as configurações principais.
