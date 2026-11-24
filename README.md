# AWS CloudFormation — EC2, Apache, Firewall, IAM e S3

Projeto desenvolvido como desafio prático de AWS CloudFormation da formação DIO. O repositório reúne quatro templates YAML em uma sequência de aprendizagem: criação de uma instância EC2, instalação do Apache, configuração de regras de firewall e composição de recursos EC2, IAM e S3 em uma única stack.

## Objetivo

Demonstrar como descrever infraestrutura como código (IaC) usando AWS CloudFormation, parametrizar recursos para reutilização e corrigir configurações que poderiam impedir a implantação ou expor serviços desnecessariamente.

## O que é AWS CloudFormation?

AWS CloudFormation é um serviço de infraestrutura como código da AWS. Com ele, os recursos são declarados em templates JSON ou YAML e provisionados como uma unidade chamada **stack**. O CloudFormation interpreta o template, cria os recursos na ordem necessária, acompanha o estado da operação e permite atualizar ou excluir a infraestrutura de forma controlada.

Em vez de configurar cada recurso manualmente, o template registra a configuração desejada e pode ser reutilizado em diferentes ambientes, com parâmetros apropriados.

## Estrutura do repositório

```text
aws-cloudformation-stacks/
├── README.md
├── templates/
│   ├── 01-ec2.yaml
│   ├── 02-apache.yaml
│   ├── 03-firewall.yaml
│   └── 04-ec2-iam-s3.yaml
└── docs/
    └── analise-dos-templates.md
```

## Templates do projeto

| Template | Recursos | O que demonstra |
|---|---|---|
| [01-ec2.yaml](templates/01-ec2.yaml) | EC2 + Security Group | Provisionamento de uma instância básica |
| [02-apache.yaml](templates/02-apache.yaml) | EC2 + UserData + Apache | Instalação e inicialização automática do servidor web |
| [03-firewall.yaml](templates/03-firewall.yaml) | EC2 + Apache + Security Group | Liberação de HTTP pela porta 80 |
| [04-ec2-iam-s3.yaml](templates/04-ec2-iam-s3.yaml) | EC2 + Security Group + IAM + S3 | Composição de recursos de diferentes serviços em uma stack |

### 1. Criando uma instância EC2

O primeiro template cria uma instância Amazon Linux 2023 em uma subnet selecionada. A AMI é resolvida por um parâmetro público do AWS Systems Manager Parameter Store, evitando manter um ID de imagem fixo no arquivo.

O Security Group não possui regras de entrada. Portanto, a instância é criada sem exposição de portas de entrada.

### 2. Instalando o Apache

O segundo template utiliza a propriedade `UserData` para executar comandos durante a inicialização da instância. O script atualiza os pacotes, instala o Apache (`httpd`), habilita o serviço e grava uma página HTML simples.

Nesta etapa, não há regra de entrada HTTP. A instalação do serviço é demonstrada separadamente da liberação de rede, que aparece no template seguinte.

### 3. Criando uma stack de Firewall

O terceiro template cria um Security Group associado à instância EC2 e permite tráfego TCP na porta 80. O servidor Apache é instalado por `UserData`.

A regra de entrada HTTP utiliza `0.0.0.0/0`, permitindo acesso público ao site. Isso é intencional para a demonstração de um servidor web público. Em ambientes reais, avalie a necessidade de restringir a origem do tráfego e adote HTTPS quando aplicável.

Para acessar o endereço exibido em `WebsiteURL`, selecione uma subnet pública com rota para um Internet Gateway e certifique-se de que a tabela de rotas e as Network ACLs não bloqueiem o tráfego.

### 4. Integrando EC2, IAM e S3

O quarto template reúne:
- Uma instância EC2 com Ubuntu 24.04 LTS, cuja AMI é consultada no Parameter Store da Canonical.
- Um Security Group que libera SSH somente para o CIDR informado no parâmetro `SSHLocation`.
- Um bucket S3 com nome gerado pela AWS, bloqueio de acesso público e criptografia padrão.
- Um grupo IAM e um usuário associado a esse grupo.

O usuário IAM e o grupo não recebem políticas de permissões neste exemplo. A criação de uma identidade não significa conceder acesso administrativo. As permissões devem ser definidas conforme o princípio do menor privilégio.

## Como criar stacks no AWS CloudFormation

1. Entre no Console de Gerenciamento da AWS e escolha a região desejada.
2. Abra o serviço **CloudFormation**.
3. Selecione **Create stack** e a opção para criar recursos com um template existente.
4. Faça upload do arquivo YAML correspondente ao exercício.
5. Informe um nome para a stack.
6. Preencha os parâmetros solicitados pelo template.
7. Revise as configurações e as permissões solicitadas.
8. Na etapa de revisão, confirme a criação da stack.
9. Acompanhe os eventos até o status de conclusão. Se houver falha, consulte os eventos da stack para identificar o recurso e a mensagem de erro.

### Parâmetros e pré-requisitos por template

**Templates 01, 02 e 03**
- Escolha uma VPC e uma subnet pertencente a essa mesma VPC.
- A AMI Amazon Linux 2023 é resolvida regionalmente pelo Parameter Store.
- Para o template 03, a subnet precisa ser pública, com rota para um Internet Gateway. O template associa um IPv4 público à instância.

**Template 04**
- Escolha uma VPC e uma subnet pública pertencente a ela.
- Informe um Key Pair existente na mesma região.
- Informe seu IP público autorizado em formato CIDR, preferencialmente com máscara `/32` (por exemplo, `203.0.113.10/32`; substitua pelo seu IP real).
- O parâmetro público da Canonical resolve a AMI Ubuntu 24.04 na região selecionada.
- O usuário e o grupo IAM precisam ter nomes disponíveis na conta.

## Cuidados com custos e segurança

- Instâncias EC2, volumes EBS, transferência de dados e outros recursos podem gerar custos. Verifique os preços e a elegibilidade do tipo de instância na região antes de criar a stack.
- Exclua as stacks de laboratório quando terminar, observando as políticas de retenção.
- O bucket S3 do template 04 tem `DeletionPolicy: Retain` e `UpdateReplacePolicy: Retain`: ele é preservado quando a stack é excluída ou substituída. A exclusão do bucket deve ser avaliada separadamente.
- O template 04 restringe SSH ao CIDR informado. Não use `0.0.0.0/0` para SSH em um ambiente real.
- O template 03 libera HTTP publicamente para fins didáticos.
- Os templates não criam usuários com privilégios administrativos nem associam políticas IAM amplas.

## O que foi corrigido e aprendido

Os exemplos foram adaptados para evitar IDs de AMI e identificadores de VPC fixos, nomes de bucket previamente ocupados, placeholders de Key Pair e regras SSH abertas para qualquer origem. Também foram atualizados os comandos de inicialização do Apache para Amazon Linux 2023 e substituído o uso de `SecurityGroups` por `SecurityGroupIds` no template de Firewall.

O projeto reforça o uso de parâmetros, referências entre recursos, outputs, UserData, Security Groups, IAM, S3, AMIs dinâmicas e boas práticas básicas de segurança.

## Validação e escopo

Os arquivos deste repositório são templates para estudo e implantação. **A publicação dos arquivos no GitHub não comprova que as stacks foram implantadas em uma conta AWS.** Registre resultados, capturas de tela e eventuais ajustes após executar os templates na sua própria conta. A implantação depende de recursos existentes e parâmetros válidos na região escolhida.

## Referências

- [AWS CloudFormation — documentação oficial](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [Tipos de parâmetros específicos da AWS no CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-supplied-parameter-types.html)
- [Amazon Linux 2023 — documentação](https://docs.aws.amazon.com/linux/al2023/ug/what-is-amazon-linux.html)
- [Canonical — localizar imagens Ubuntu na AWS](https://ubuntu.com/aws/docs/aws-how-to/instances/find-ubuntu-images/)
- [AWS::EC2::SecurityGroup — referência](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-resource-ec2-securitygroup.html)
