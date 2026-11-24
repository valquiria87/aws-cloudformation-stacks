# Análise técnica e correções dos templates

Este documento registra as principais adaptações feitas nos exemplos para aumentar a portabilidade, reduzir configurações frágeis e evitar exposições desnecessárias.

## Template 01 — EC2

- Removido o ID de AMI fixo, substituído por parâmetro público do Systems Manager para Amazon Linux 2023.
- Removida a dependência de uma Availability Zone fixa.
- Adicionados parâmetros para selecionar VPC e subnet.
- Criado Security Group sem regras de entrada, evitando expor portas que não fazem parte deste exercício.
- Incluídos Outputs para consultar ID, IP privado e Security Group.

## Template 02 — Apache

- Mantida a demonstração de instalação automática por UserData.
- Atualizada a instalação para Amazon Linux 2023, usando `dnf`.
- A AMI é resolvida regionalmente pelo Parameter Store.
- VPC e subnet são parâmetros.
- Não foi aberta a porta 80 nesta etapa: a instalação do Apache e a regra de firewall ficam separadas para demonstrar a evolução entre os templates.

## Template 03 — Firewall

- Substituído o atributo legado `SecurityGroups` por `SecurityGroupIds`.
- O Security Group declara explicitamente a VPC.
- A instância usa uma subnet selecionada e recebe IPv4 público.
- A regra TCP/80 é aberta para `0.0.0.0/0`, pois o objetivo é demonstrar um servidor HTTP público. Essa origem deve ser reavaliada em ambientes reais.
- A subnet precisa ter rota para um Internet Gateway para que o site seja alcançável pela internet.

## Template 04 — EC2, IAM e S3

- O nome fixo do bucket foi removido; a AWS gera um nome físico exclusivo.
- O ID da VPC foi substituído por parâmetro do tipo `AWS::EC2::VPC::Id`.
- A subnet é selecionada por parâmetro.
- O placeholder do Key Pair foi substituído por parâmetro do tipo `AWS::EC2::KeyPair::KeyName`.
- O ID de AMI fixo foi substituído por parâmetro público SSM da Canonical para Ubuntu 24.04 LTS.
- A regra SSH usa o CIDR informado em `SSHLocation`, em vez de permitir acesso de qualquer IP.
- O bucket bloqueia acesso público, usa criptografia SSE-S3 e possui política de retenção.
- O grupo e o usuário IAM são criados sem políticas de permissões administrativas.

## Dependências que precisam existir na conta

- Uma VPC e uma subnet compatível na mesma região para cada template.
- Para os templates 03 e 04, uma subnet pública com rota para Internet Gateway, caso seja necessário acesso externo ao servidor.
- Um Key Pair existente na região para o template 04.
- Um CIDR de origem autorizado para SSH no template 04.
- Nomes IAM disponíveis na conta para os recursos nomeados do template 04.

## Limites da validação

As correções foram aplicadas aos arquivos YAML e publicadas no repositório. Isso não substitui a validação pelo CloudFormation nem uma implantação real. A confirmação final depende de criar as stacks na região escolhida e acompanhar os eventos. Não se deve afirmar que houve implantação bem-sucedida sem esse teste.
