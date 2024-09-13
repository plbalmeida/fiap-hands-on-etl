# Provisionamento dos recursos com Terraform

## Role do IAM

Seguindo a ordem de criação dos recursos vista na parte 2 desse hands-on, começamos pela role do IAM.

Na raíz do diretório de repositório git, criamos a pasta `ìam`:

```bash
mkdir iam
```

E criamos os seguintes arquivos, `providers.tf` e `main.tf` respectivamente:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

```hcl
resource "aws_iam_role" "fiap_etl_role" {
  name = "fiap-etl-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = "sts:AssumeRole"
        Principal = {
          AWS = "413467296690"
        }
      },
      {
        Effect = "Allow"
        Action = "sts:AssumeRole"
        Principal = {
          Service = [
            "events.amazonaws.com",
            "states.amazonaws.com",
            "glue.amazonaws.com"
          ]
        }
      }
    ]
  })
}

resource "aws_iam_policy_attachment" "power_user_access" {
  name       = "attach-power-user-access"
  roles      = [aws_iam_role.fiap_etl_role.name]
  policy_arn = "arn:aws:iam::aws:policy/PowerUserAccess"
}

```

Vá até o diretório `iam` e execute os seguinte comandos para provisionar o recurso:

```bash
cd iam
terraform init
terraform plan
terraform apply
```

## S3

Criamos os buckets.

Na raíz do diretório de repositório git, criamos a pasta `s3`:

```bash
mkdir s3
```

E criamos os seguintes arquivos, `providers.tf` e `main.tf`:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

```hcl
resource "aws_s3_bucket" "fiap_etl_bucket" {
  bucket = "fiap-etl"
}

resource "aws_s3_object" "raw_path" {
  bucket = aws_s3_bucket.fiap_etl_bucket.bucket
  key    = "raw/"
}

resource "aws_s3_object" "interim_path" {
  bucket = aws_s3_bucket.fiap_etl_bucket.bucket
  key    = "interim/"
}

resource "aws_s3_object" "final_path" {
  bucket = aws_s3_bucket.fiap_etl_bucket.bucket
  key    = "final/"
}

resource "aws_s3_object" "query_results_path" {
  bucket = aws_s3_bucket.fiap_etl_bucket.bucket
  key    = "query-results/"
}

resource "aws_s3_object" "scripts_path" {
  bucket = aws_s3_bucket.fiap_etl_bucket.bucket
  key    = "scripts/"
}

```

Vá até o diretório `s3` e execute os seguinte comandos para provisionar o recurso:

```bash
cd s3
terraform init
terraform plan
terraform apply
```

## Glue

Agora vamos criar os jobs do AWS Glue.

Na raíz do diretório de repositório git, criamos a pasta `glue`:

```bash
mkdir glue
```

E criamos os seguintes arquivos, `providers.tf`, `main.tf` e `variables.tf` respectivamente:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

```hcl
# role que será usada nos jobs
data "aws_iam_role" "glue_role" {
  name = var.role_name
}

resource "aws_glue_job" "extract_job" {
  name     = "extract"
  role_arn = data.aws_iam_role.glue_role.arn

  command {
    name            = "glueetl"
    python_version  = "3"
    script_location = "${var.script_location}extract.py"
  }

  default_arguments = var.default_arguments

  max_retries       = 0
  worker_type       = "G.1X"
  number_of_workers = 2
}

resource "aws_glue_job" "transform_job" {
  name     = "transform"
  role_arn = data.aws_iam_role.glue_role.arn

  command {
    name            = "glueetl"
    python_version  = "3"
    script_location = "${var.script_location}transform.py"
  }

  default_arguments = var.default_arguments

  max_retries       = 0
  worker_type       = "G.1X"
  number_of_workers = 2
}

resource "aws_glue_job" "load_job" {
  name     = "load"
  role_arn = data.aws_iam_role.glue_role.arn

  command {
    name            = "glueetl"
    python_version  = "3"
    script_location = "${var.script_location}load.py"
  }

  default_arguments = var.default_arguments

  max_retries       = 0
  worker_type       = "G.1X"
  number_of_workers = 2
}

```

```hcl
variable "role_name" {
  default = "fiap-etl-role"
}

variable "script_location" {
  default = "s3://fiap-etl/scripts/"
}

variable "default_arguments" {
  type = map(string)
  default = {
    "--job-bookmark-option"       = "job-bookmark-disable"
    "--enable-glue-datacatalog"   = "true"
    "--additional-python-modules" = "ipeadatapy"
  }
}
```

Vá até o diretório `glue` e execute os seguinte comandos para provisionar o recurso:

```bash
cd glue
terraform init
terraform plan
terraform apply
```

## Step Functions

Agora vamos criar a máquina de estado do Step Functions que irá orquestrar a execução dos jobs do AWS Glue.

Na raíz do diretório de repositório git, criamos a pasta `stepfunctions`:

```bash
mkdir stepfunctions
```

E criamos os seguintes arquivos, `providers.tf` e `main.tf` respectivamente:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

```hcl
# referência para a role IAM
data "aws_iam_role" "fiap_etl_role" {
  name = "fiap-etl-role"
}

# definição da máquina de estado do Step Functions
resource "aws_sfn_state_machine" "fiap_etl" {
  name     = "fiap-etl"
  role_arn = data.aws_iam_role.fiap_etl_role.arn

  definition = jsonencode({
    "Comment" : "Orquestração dos jobs de ETL usando Step Functions",
    "StartAt" : "ExtractJob",
    "States" : {
      "ExtractJob" : {
        "Type" : "Task",
        "Resource" : "arn:aws:states:::glue:startJobRun.sync",
        "Parameters" : {
          "JobName" : "extract"
        },
        "Catch" : [{
          "ErrorEquals" : ["States.ALL"],
          "Next" : "FailState"
        }],
        "Next" : "TransformJob"
      },
      "TransformJob" : {
        "Type" : "Task",
        "Resource" : "arn:aws:states:::glue:startJobRun.sync",
        "Parameters" : {
          "JobName" : "transform"
        },
        "Catch" : [{
          "ErrorEquals" : ["States.ALL"],
          "Next" : "FailState"
        }],
        "Next" : "LoadJob"
      },
      "LoadJob" : {
        "Type" : "Task",
        "Resource" : "arn:aws:states:::glue:startJobRun.sync",
        "Parameters" : {
          "JobName" : "load"
        },
        "Catch" : [{
          "ErrorEquals" : ["States.ALL"],
          "Next" : "FailState"
        }],
        "End" : true
      },
      "FailState" : {
        "Type" : "Fail",
        "Error" : "JobFailed",
        "Cause" : "A execução do job falhou."
      }
    }
  })
}

```

Vá até o diretório `stepfunctions` e execute os seguinte comandos para provisionar o recurso:

```bash
cd stepfunctions
terraform init
terraform plan
terraform apply
```

## EventBridge

Por fim, criaremos uma `rule` do EventBridge para disparar a execução da máquina de estado do Step Functions de maneira agendada.

Na raíz do diretório de repositório git, criamos a pasta `eventbridge`:

```bash
mkdir eventbridge
```

E criamos os seguintes arquivos, `providers.tf` e `main.tf` respectivamente:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

```hcl
# obtém a role existente fiap-etl-role
data "aws_iam_role" "fiap_etl_role" {
  name = "fiap-etl-role"
}

# obtém a máquina de estado previamente criada
data "aws_sfn_state_machine" "fiap_etl_state_machine" {
  name = "fiap-etl"
}

# regra do EventBridge para disparar o Step Functions diariamente
resource "aws_cloudwatch_event_rule" "daily_fiap_etl" {
  name                = "fiap-etl-rule"
  description         = "Regra do EventBridge para executar a máquina de estado fiap-etl diariamente às 18:30 UTC"
  schedule_expression = "cron(30 18 * * ? *)"
}

# permissão para o EventBridge invocar o Step Functions
resource "aws_cloudwatch_event_target" "target_step_functions" {
  rule     = aws_cloudwatch_event_rule.daily_fiap_etl.name
  arn      = data.aws_sfn_state_machine.fiap_etl_state_machine.arn
  role_arn = data.aws_iam_role.fiap_etl_role.arn

  input = jsonencode({})
}

```

Vá até o diretório `eventbridge` e execute os seguinte comandos para provisionar o recurso:

```bash
cd eventbridge
terraform init
terraform plan
terraform apply
```

Para testar a execução, ajuste o CRON para uma hora próxima.

## .gitignore

Caso for utilizar o GitHub, na raíz do repositório crie o arquivo `.gitignore` para não fazer o push dos seguintes arquivos do Terraform:

```.gitignore
.terraform
.terraform.lock.hcl
terraform.tfstate
terraform.tfstate.backup
```

## Destruição dos recursos

Para destruir os recursos provisionados:

```bash
cd <PATH>
terraform destroy
```

## Repositório da parte 3

O exercício está disponível no seguinte link: [https://github.com/plbalmeida/fiap-etl-terraform](https://github.com/plbalmeida/fiap-etl-terraform)
