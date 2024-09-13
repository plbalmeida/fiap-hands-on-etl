# Set up do ambiente para a parte 3

Aqui é apresentados o passo a passo para configurar o ambiente para usar o Terraform, incluindo a instalação do Terraform e a configuração das credenciais da AWS como variáveis de ambiente locais.

## Instalação do Terraform

### Requisitos

- **Sistema Operacional**: Windows, macOS ou Linux
- **Permissões de Administrador**: Necessárias para instalar o Terraform no sistema

### Passos de Instalação

1. **Download do Terraform**:

   Baixe a versão mais recente do Terraform a partir do site oficial:

   - Página de Downloads do Terraform: https://www.terraform.io/downloads.html

2. **Instalação**:

   - **Linux**:

     - Extraia o arquivo ZIP baixado.
     - Mova o binário `terraform` para `/usr/local/bin/`:

       ```bash
       sudo mv terraform /usr/local/bin/
       ```

3. **Verificação da Instalação**:

   Abra o terminal e execute:

   ```bash
   terraform -v
   ```

   Você deverá ver a versão instalada do Terraform.

Para mais detalhes, consulte a documentação oficial:

- Guia de Instalação do Terraform: https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli

## Configuração das Credenciais da AWS

Para que o Terraform possa interagir com a AWS, é necessário configurar suas credenciais de acesso.

### Obter Credenciais da AWS

- Acesse o **Console de Gerenciamento da AWS**: https://console.aws.amazon.com/.
- Navegue até o **IAM (Gerenciamento de Identidade e Acesso)**: (https://console.aws.amazon.com/iam/).
- Crie um novo usuário ou utilize um existente com permissões adequadas.
- Gere uma **Access Key ID** e uma **Secret Access Key**.

### Definir Variáveis de Ambiente

Defina as seguintes variáveis de ambiente no seu sistema:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- (Opcional) `AWS_DEFAULT_REGION`

#### Linux

No terminal bash:

```bash
export AWS_ACCESS_KEY_ID="<SUA ACCESS KEY ID>"
export AWS_SECRET_ACCESS_KEY="<SUA SECRET ACCESS KEY>"
export AWS_DEFAULT_REGION="<REGIÃO>"
```

## Recursos Adicionais

- Documentação Oficial do Terraform sobre o Provedor AWS: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
- Guia de Configuração de Credenciais da AWS: https://docs.aws.amazon.com/pt_br/cli/latest/userguide/cli-configure-envvars.html
- Tutorial de Introdução ao Terraform com AWS: https://learn.hashicorp.com/tutorials/terraform/aws-build