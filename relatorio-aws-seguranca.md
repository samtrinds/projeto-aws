## Relatório de implementação de medidas de segurança 

Data: 28/10/2025
Empresa: Abstergo Industries
Responsável: Samara da Trindade

## Introdução

A Abstergo Industries, farmácia digital em expansão, vem aprimorando sua infraestrutura em nuvem com foco em desempenho, disponibilidade e segurança. No primeiro projeto, foi criado um ambiente completo na AWS utilizando EC2, S3, ALB e CloudWatch, o que garantiu alta disponibilidade e escalabilidade para o site da empresa.
Com o ambiente já estruturado, o próximo passo é reforçar a segurança das aplicações, da conta AWS e das informações sensíveis usadas em produção. Este relatório propõe três ferramentas da AWS que podem ser implementadas para fortalecer a segurança da infraestrutura existente, composta por EC2, S3, ALB e CloudWatch.



## Descrição do projeto

As soluções que foram escolhidas para abranger três camadas principais de proteção:

`Segurança de dados e variáveis sensíveis`
`Segurança do site e da aplicação web`
`Segurança e conformidade da conta AWS`

## Medidas 

1) AWS Key Management Service (KMS)

O AWS Key Management Service (KMS) é o serviço responsável por criar e gerenciar chaves de criptografia dentro da AWS. Ele permite proteger dados armazenados em diversos serviços e garantir que apenas recursos autorizados tenham acesso a informações sigilosas. 
Na Abstergo Industries, o KMS será usado para proteger **variáveis de ambiente e credenciais sensíveis, como senhas, tokens de APIs e chaves de produção**.
Em muitos casos, essas informações ficam armazenadas em arquivos `.env` dentro do código do site, o que pode representar um risco grave se o repositório for exposto. Um caso real envolveu um teste de invasão em que um site hospedado na AWS teve suas variáveis expostas no corpo da página, revelando credenciais de acesso.
Usando o KMS integrado ao AWS Secrets Manager ou Systems Manager Parameter Store, essas variáveis são criptografadas e descriptografadas apenas quando o sistema autorizado solicita. 

2) AWS Web Application Firewall (WAF)

O AWS WAF é uma camada de proteção voltada para sites e aplicações web. Ele atua em conjunto com o Application Load Balancer (ALB), filtrando todo o tráfego que chega antes de atingir a instância EC2. O serviço analisa as requisições e bloqueia ataques conhecidos, como SQL Injection, Cross-Site Scripting (XSS) e tentativas de exploração de vulnerabilidades.
Além disso, o WAF permite criar regras personalizadas para bloquear padrões suspeitos, e também disponibiliza regras gerenciadas mantidas pela própria AWS. Na Abstergo, o WAF será configurado sobre o ALB para proteger o site e os formulários da farmácia, impedindo que códigos maliciosos cheguem ao servidor.

3) AWS Security Hub

O AWS Security Hub fornece uma visão completa da segurança da conta e ajuda a garantir que todas as boas práticas da AWS estejam sendo seguidas. Ele coleta e centraliza resultados de vários serviços de segurança, como GuardDuty, Config, IAM e CloudTrail, em um único painel de controle.
Na Abstergo Industries, o Security Hub permitirá auditar continuamente o ambiente, identificando usuários sem MFA, permissões excessivas, buckets públicos e instâncias vulneráveis. Com isso, será possível agir rapidamente antes que essas falhas se tornem brechas reais.
O painel também exibe recomendações automáticas com base nos padrões de conformidade da AWS, ajudando a manter a conta sempre protegida.

## Conclusão 

A adoção do KMS, WAF e Security Hub representa uma evolução na segurança da Abstergo Industries dentro da nuvem AWS. Essas três ferramentas se complementam e formam uma camada extra de proteção, pois o KMS protege os dados e variáveis sigilosas, WAF protege o site e a camada de aplicação e o Security Hub supervisiona todo o ambiente em busca de falhas e más práticas.

O documento `implementando-segurança` detalhará o passo a passo de implementação dessas três ferramentas dentro do ambiente existente, mantendo o uso do Free Tier e as boas práticas de segurança em nuvem.
