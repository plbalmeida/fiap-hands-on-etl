Esse hands-on possui 4 partes.

A **Parte 1** começa com dados do IPEA para desenvolvimento de ETL para construção de features:

- **Nivelamento técnico**: Visão geral sobre ETL no contexto de Engenharia de Machine Learning.
  
- **Engenharia de features**: Features mais comuns extraídas de séries temporais com engenharia de features.
  
- **ETL com pyspark**: Implementação de ETL com PySpark.

Na **Parte 2** o foco será subir o processo de ETL construído anteriormente na AWS:

- **Buckets do S3**: Configuração de buckets no Amazon S3 para armazenar os dados brutos e processados, com foco em boas práticas de segurança e organização.

- **AWS Glue**: Criação e configuração de jobs no AWS Glue para automatizar o processo de ETL, aproveitando a integração com S3 e a escalabilidade do serviço.

- **Orquestração de ETL com AWS Step Functions**: Configuração de um fluxo de trabalho usando AWS Step Functions para orquestrar o pipeline de ETL de ponta a ponta.

Na **Parte 3** faremos o provisionamento dos recursos da AWS provisionados anteriormente usando IaC com Terraform:

- **Visão geral sobre Terraform**: Introdução ao Terraform, explicando os conceitos básicos e como ele facilita o gerenciamento de infraestrutura como código.

- **IaC com Terraform**: Implementação prática de scripts Terraform para provisionar os recursos da AWS, como S3, Glue, e Step Functions, garantindo reprodutibilidade e controle de versão.

Na **Parte 4** criaremos um pipeline de CI/CD para automação de testes e deploy dos recursos usados na AWS usando GitHub Actions:

- **Visão geral sobre CI/CD e GitHub Actions**: Conceitos fundamentais de integração e entrega contínua, e como o GitHub Actions pode ser utilizado para automatizar pipelines de DevOps.

- **CI/CD de ETL com GitHub Actions**: Implementação de um pipeline de CI/CD para o ETL na AWS, incluindo testes automatizados e deploy na AWS.
```
