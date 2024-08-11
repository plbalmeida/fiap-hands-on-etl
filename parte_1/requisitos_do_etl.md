# Requisitos do ETL

## Desenvolvimento do Processo de ETL no Contexto de Engenharia de Machine Learning

O desenvolvimento de um processo de ETL (Extract, Transform, Load) no contexto de Engenharia de Machine Learning envolve uma série de etapas que começam com o levantamento de requisitos junto aos stakeholders, como Cientistas de Dados, pessoas de negócios e outros profissionais relevantes. Este levantamento é essencial para garantir que os dados coletados, transformados e carregados atendam às necessidades específicas dos modelos de Machine Learning e dos objetivos de negócio.

### Levantamento de Requisitos com Stakeholders

O processo de levantamento de requisitos é uma fase crítica em qualquer projeto de ETL. Nesse contexto, é fundamental entender as expectativas e necessidades dos Cientistas de Dados, que utilizarão os dados transformados para criar modelos preditivos, bem como das pessoas de negócios, que podem estar interessadas em como esses modelos impactarão as decisões estratégicas da empresa.

Durante as reuniões com esses stakeholders, é importante esclarecer:

- **Objetivos do Modelo de ML**: Qual é o objetivo principal do modelo que será treinado? Quais tipos de previsões ou insights são esperados?
- **Fonte dos Dados**: De onde vêm os dados? Quais fontes de dados precisam ser integradas?
- **Formato e Estrutura dos Dados**: Quais são os requisitos de formatação dos dados? Existe alguma estrutura específica necessária para o modelo?
- **Engenharia de Features**: Quais features precisam ser extraídas ou criadas a partir dos dados brutos? Quais são as variáveis mais relevantes?
- **Qualidade dos Dados**: Quais critérios de qualidade precisam ser atendidos? Existem limitações ou considerações específicas, como a necessidade de lidar com dados faltantes ou inconsistentes?
- **Escalabilidade e Manutenção**: Como o processo de ETL será mantido e escalado ao longo do tempo? Há necessidade de automatização e orquestração?

Ao coletar essas informações, é possível definir de forma clara o escopo e os requisitos do processo de ETL, assegurando que ele atenda às necessidades tanto técnicas quanto de negócios.

## Descrição dos Requisitos dos Dados

Com base no levantamento de requisitos, é possível construir um catálogo de dados que descreva em detalhes as necessidades específicas para o processo de ETL.

> No caso específico deste hands-on, vamos supor que o Cientista de Dados do seu time tem como objetivo prever a média mensal do preço do petróleo bruto do IPEA. Para isso, os seguintes campos com os respectivos tipos foram solicitados. Ainda foi pedido para que os dados do mês vigente não sejam calculados.

Dado o levantamento de requisito da tabela final, foi documentado o seguinte dicionário de dados, alinhando os nomes dos campos na tabela, sua descrição e tipagem dos dados. 

| Nome da Variável                              | Descrição                                                 | Tipo de Dado | Formato      |
|-----------------------------------------------|-----------------------------------------------------------|--------------|--------------|
| `ano`                                         | Ano da observação                                         | Integer      | integer      |
| `mes`                                         | Mês da observação                                         | Integer      | integer      |
| `preco_medio_usd`                             | Média mensal do preço do petróleo bruto                   | Decimal      | decimal(5,2) |
| `lag_1_mes_preco_medio_usd`                   | Defasagem de 1 mês da média mensal do preço do petróleo   | Decimal      | decimal(5,2) |
| `lag_2_mes_preco_medio_usd`                   | Defasagem de 2 meses da média mensal do preço do petróleo | Decimal      | decimal(5,2) |
| `lag_3_mes_preco_medio_usd`                   | Defasagem de 3 meses da média mensal do preço do petróleo | Decimal      | decimal(5,2) |
| `lag_4_mes_preco_medio_usd`                   | Defasagem de 4 meses da média mensal do preço do petróleo | Decimal      | decimal(5,2) |
| `lag_5_mes_preco_medio_usd`                   | Defasagem de 5 meses da média mensal do preço do petróleo | Decimal      | decimal(5,2) |
| `lag_6_mes_preco_medio_usd`                   | Defasagem de 6 meses da média mensal do preço do petróleo | Decimal      | decimal(5,2) |
| `media_movel_6_meses_preco_medio_usd`         | Média móvel dos últimos 6 meses do preço do petróleo      | Decimal      | decimal(5,2) |
| `desvio_padrao_movel_6_meses_preco_medio_usd` | Desvio padrão móvel dos últimos 6 meses                   | Decimal      | decimal(5,2) |
| `valor_minimo_6_meses_preco_medio_usd`        | Valor mínimo dos últimos 6 meses                          | Decimal      | decimal(5,2) |
| `valor_maximo_6_meses_preco_medio_usd`        | Valor máximo dos últimos 6 meses                          | Decimal      | decimal(5,2) |
| `trimestre`                                   | Trimestre da observação                                   | Integer      | integer      |

### Detalhes dos Requisitos dos Dados

1. **Média Mensal do Petróleo Bruto**: Este campo representa a média mensal do preço do petróleo bruto e é uma das variáveis mais importantes para o modelo preditivo.

2. **Variáveis de Lags (1 a 6 meses)**: Lags são usadas para capturar a defasagem temporal e entender como o preço passado pode influenciar as previsões futuras. Cada variável de lag é criada a partir da média mensal do petróleo bruto.

3. **Estatísticas Móveis (Média, Desvio Padrão, Mínimo, Máximo)**: Essas variáveis fornecem uma visão agregada do comportamento do preço do petróleo ao longo dos últimos 6 meses, permitindo que o modelo capture tendências e volatilidade.

4. **Variáveis de Mês e Trimestre**: Essas variáveis sazonais ajudam a capturar padrões temporais relacionados ao mês e ao trimestre em que a observação foi feita.

## Conclusão

O levantamento de requisitos e a definição clara das variáveis necessárias são etapas fundamentais para o sucesso de qualquer processo de ETL em Engenharia de Machine Learning. Ao criar um catálogo de dados detalhado, garantimos que todos os stakeholders estejam alinhados e que o processo de ETL seja construído para atender às necessidades do projeto, resultando em dados de alta qualidade prontos para alimentar modelos preditivos.
