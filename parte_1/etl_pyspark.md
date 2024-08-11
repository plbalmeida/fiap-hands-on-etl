# Engenharia de Features com ETL

A engenharia de features é o processo de criar novas variáveis relevantes a partir de um conjunto de dados original que possam ser utilizadas por algoritmos de ML para melhorar a sua performance de estimação. Este processo envolve a criação, seleção e modificação de variáveis a partir dos dados disponíveis, visando representar de forma mais eficaz os padrões subjacentes e as relações importantes entre os dados. Na prática, isso pode incluir operações como normalização, criação de novas variáveis a partir de combinações das existentes, extração de estatísticas agregadas e tratamento de valores ausentes. Uma boa engenharia de features é essencial para o sucesso de qualquer projeto de ML, pois impacta diretamente a capacidade do modelo de aprender e fazer previsões precisas.

## Engenharia de Features para Série Temporal

A engenharia de features é um passo crucial na preparação dos dados de séries temporais para modelos de ML. A seguir, é listada algumas das features mais comuns extraídas de séries temporais:

- **Lags (Defasagens)**: Valores anteriores da série temporal são usados como características.
- **Médias Móveis**: Média dos valores anteriores para suavizar a série e capturar tendências.
- **Diferenças**: Diferença entre valores consecutivos para remover tendências e tornar a série estacionária.
- **Componentes Sazonais**: Extração de padrões sazonais que se repetem em intervalos regulares.
- **Transformações Estatísticas**: Desvios padrão, variação e outras medidas estatísticas ao longo do tempo.
- **Features Calendáricas**: Dia da semana, mês, feriados e outras variáveis temporais.

A engenharia de features melhora a capacidade do modelo de capturar padrões temporais complexos e aumenta a precisão das previsões.

## Implementação do ETL com PySpark

A seguir serão obtidas as features para a tabela demandada no levantamento de requisitos junto dos stakeholders do projeto usando o PySpark.

PySpark é uma interface para Apache Spark em Python. Spark é uma poderosa ferramenta de computação distribuída usada para processar grandes volumes de dados de maneira eficiente, especialmente em clusters de computadores.

PySpark inclui várias bibliotecas que facilitam essas operações, no presente projeto utilizaremos o `DataFrame` do PySpark, que similar aos DataFrames do Pandas, é uma abstração para dados estruturados, mas distribuída e otimizada para grandes volumes de dados.

Link da documentação do PySpark: [https://spark.apache.org/docs/3.1.1/api/python/index.html](https://spark.apache.org/docs/3.1.1/api/python/index.html)

### 1. Etapa de "Extract"

Nesta etapa, vamos extrair os dados brutos de uma fonte, neste caso, assumimos que já temos um DataFrame `df` carregado. Vamos criar uma pasta chamada `data/raw` e salvar os dados no formato `.parquet`.

```python
import ipeadatapy as ip
from pyspark.sql import SparkSession
from pyspark.sql.functions import current_timestamp, date_format

# iniciando uma sessão do Spark
spark = SparkSession.builder \
    .appName("ETL Example") \
    .getOrCreate()

# obtenção da série temporal do preço do petróleo bruto
cod = "EIA366_PBRENT366"
eia366 = ip.timeseries(cod)

# conversão de pandas DataFrame para o DataFrame do PySpark
df = spark.createDataFrame(eia366)

# diretório para salvar os dados brutos
raw_data_path = "../data/raw"

# dados brutos em formato Parquet
df.write.mode("overwrite").parquet(raw_data_path)
```

### 2. Etapa de "Transform"

Na etapa de transformação, vamos aplicar a engenharia de features ao DataFrame. Após as transformações, vamos salvar o resultado em uma nova pasta, por exemplo, data/transformed.

```python
from pyspark.sql.functions import (
    avg, 
    col, 
    concat,
    current_date,
    date_format,
    lag,
    lpad,
    max,
    min,
    quarter,
    stddev,
    to_date
)
from pyspark.sql.window import Window

# carregando os dados brutos
raw_data_path = "../data/raw"
df = spark.read.parquet(raw_data_path)

# obtendo o ano e mês corrente
current_year = date_format(current_date(), "yyyy").cast("int")
current_month = date_format(current_date(), "MM").cast("int")

# filtrando para remover os dados do mês corrente
df = df.filter(~((col("YEAR") == current_year) & (col("MONTH") == current_month)))

# removendo registros com valores ausentes em "VALUE (US$)"
df = df.na.drop(subset=["VALUE (US$)"])

# média mensal de preço do petróleo bruto
df = df.groupBy("YEAR", "MONTH").agg(avg("VALUE (US$)").alias("preco_medio_usd"))

# criação da coluna "anomes" utilizando os campos "YEAR" e "MONTH"
df = df.withColumn("anomes", to_date(concat(col("YEAR").cast("string"), lpad(col("MONTH").cast("string"), 2, "0")), "yyyyMM"))

# ordenando o DataFrame por data
window_spec = Window.orderBy("anomes")

# criando lags de 1 a 6 meses
df = df.withColumn("lag_1_mes_preco_medio_usd", lag("preco_medio_usd", 1).over(window_spec)) \
       .withColumn("lag_2_meses_preco_medio_usd", lag("preco_medio_usd", 2).over(window_spec)) \
       .withColumn("lag_3_meses_preco_medio_usd", lag("preco_medio_usd", 3).over(window_spec)) \
       .withColumn("lag_4_meses_preco_medio_usd", lag("preco_medio_usd", 4).over(window_spec)) \
       .withColumn("lag_5_meses_preco_medio_usd", lag("preco_medio_usd", 5).over(window_spec)) \
       .withColumn("lag_6_meses_preco_medio_usd", lag("preco_medio_usd", 6).over(window_spec))

# média móvel de 6 meses
df = df.withColumn("media_movel_6_meses_preco_medio_usd", avg(col("preco_medio_usd")).over(window_spec.rowsBetween(-6, -1)))

# desvio padrão móvel de 6 meses
df = df.withColumn("desvio_padrao_movel_6_meses_preco_medio_usd", stddev(col("preco_medio_usd")).over(window_spec.rowsBetween(-6, -1)))

# valor mínimo e máximo dos últimos 6 meses
df = df.withColumn("valor_minimo_6_meses_preco_medio_usd", min(col("preco_medio_usd")).over(window_spec.rowsBetween(-6, -1))) \
       .withColumn("valor_maximo_6_meses_preco_medio_usd", max(col("preco_medio_usd")).over(window_spec.rowsBetween(-6, -1)))

# componentes sazonais: ano, mês e trimestre
df = df.withColumn("trimestre", quarter("anomes"))
df = df.withColumnRenamed("YEAR", "ano")
df = df.withColumnRenamed("MONTH", "mes")

# remoção do campo "anomes"
df = df.drop("anomes")

# removendo linhas com valores NaN que foram criados ao fazer o shift ou nas agregações
df = df.dropna()

# persistindo os dados transformados
transformed_data_path = "../data/transformed"
df.write.mode("overwrite").parquet(transformed_data_path)
```

### 3. Etapa de "Load"

Finalmente, na etapa de "Load", vamos carregar os dados transformados e forçar os tipos de dados necessários antes de salvar a versão final no formato .parquet.

```python
from pyspark.sql.types import DecimalType, IntegerType

# carregando os dados transformados
transformed_data_path = "../data/transformed"
df = spark.read.parquet(transformed_data_path)

# forçando os tipos de dados
df = df.select(
    col("ano").cast(IntegerType()),
    col("mes").cast(IntegerType()),
    col("preco_medio_usd").cast(DecimalType(5,2)),
    col("lag_1_mes_preco_medio_usd").cast(DecimalType(5,2)),
    col("lag_2_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("lag_3_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("lag_4_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("lag_5_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("lag_6_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("media_movel_6_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("desvio_padrao_movel_6_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("valor_minimo_6_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("valor_maximo_6_meses_preco_medio_usd").cast(DecimalType(5,2)),
    col("trimestre").cast(IntegerType())
)

# persistindo os dados prontos para consumo
final_data_path = "../data/final"
df.write.mode("overwrite").parquet(final_data_path)
```

Na próxima parte desse hands-on, vamos continuar avançando na criação deste pipeline de dados, desta vez integrando-o com a infraestrutura da AWS para que possamos escalar nossas operações e aproveitar o poder da nuvem para orquestrar e automatizar o processo de ETL.