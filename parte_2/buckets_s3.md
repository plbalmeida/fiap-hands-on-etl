# Buckets S3

Agora criaremos o *bucket* e respectivos paths que serão usados nesse hands-on, no console do S3 clicamos no botão *Create bucket*, o nome do *bucket* será `fiap-etl`:

<div align="center">
  <figure>
    <img src="s3_1.png" alt="">
    <figcaption>
      Fonte: Elaboração própria
    </figcaption>
  </figure>
</div>

Com o bucket criado, acessar o mesmo e criar os diretórios, `raw`, `interim`, `final` e `query-results` clicando no botão *Create folder*:

<div align="center">
  <figure>
    <img src="s3_2.png" alt="">
    <figcaption>
      Fonte: Elaboração própria
    </figcaption>
  </figure>
</div>

É esperado os paths no bucket:

<div align="center">
  <figure>
    <img src="s3_3.png" alt="">
    <figcaption>
      Fonte: Elaboração própria
    </figcaption>
  </figure>
</div>

O path `query-results` será utilizado pelo Athena para armazenar as queries.