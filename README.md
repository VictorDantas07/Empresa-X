# Empresa-X

# Análise - Inovação Cashback

Utilizando as bases fícticias que estão contidas nesse repositório, temos o seguinte código para criação das tabelas no banco de dados PostgreSQL
```
CREATE TABLE cliente (
    id_cliente VARCHAR(255),
	id_unique_cliente VARCHAR(255),
	cliente_zip_code_prefix INT,
	cliente_cidade VARCHAR(255),
	cliente_uf VARCHAR(2)
);

COPY      cliente(id_cliente, id_unique_cliente, cliente_zip_code_prefix, cliente_cidade, cliente_uf)
FROM      'C:\olist_customers_dataset.csv'
DELIMITER ','
CSV HEADER;

CREATE TABLE itens (
    id_compra VARCHAR(255),
	id_item_compra VARCHAR(255),
	id_produto VARCHAR(255),
	id_vendedor VARCHAR(255),	
	shipping_limit_date timestamp,
	preco DECIMAL(10,2),
	frete DECIMAL(10,2)
);

DROP TABLE itens
SELECT * FROM itens

COPY      itens(id_compra, id_item_compra, id_produto, id_vendedor, shipping_limit_date, preco, frete)
FROM      'C:\olist_order_items_dataset.csv'
DELIMITER ','
CSV HEADER;

CREATE TABLE pagamento (
    id_compra VARCHAR(255),
    sequencia_compra BIGINT,	
	tipo_pagamento VARCHAR(255),
	qtde_parcelas BIGINT,
	vlr_pagamento DECIMAL(10,2)
);

DROP TABLE olist_customers_dataset

COPY      pagamento(id_compra, sequencia_compra, tipo_pagamento, qtde_parcelas, vlr_pagamento)
FROM      'C:\olist_order_payments_dataset.csv'
DELIMITER ','
CSV HEADER;


CREATE TABLE compra (
    id_compra VARCHAR(255),
	id_cliente VARCHAR(255),
	status_compra VARCHAR(255),
	order_purchase_timestamp TIMESTAMP,
	dt_compra TIMESTAMP,
	order_delivered_carrier_date TIMESTAMP,
    order_delivered_customer_date TIMESTAMP,
	order_estimated_delivery_date TIMESTAMP
);

COPY      compra(id_compra, id_cliente, status_compra, order_purchase_timestamp, dt_compra, order_delivered_carrier_date, order_delivered_customer_date, order_estimated_delivery_date)
FROM      'C:\olist_orders_dataset.csv'
DELIMITER ','
CSV HEADER;


CREATE TABLE produtos (
    id_produto VARCHAR(255),
	produto_categoria VARCHAR(255),
	product_name_lenght INT,
	product_description_lenght INT,
	product_photos_qty INT,
	product_weight_g INT,
	product_length_cm INT,
	product_height_cm INT,
	product_width_cm INT
);

COPY      produtos(id_produto, produto_categoria, product_name_lenght, product_description_lenght, product_photos_qty, product_weight_g, product_length_cm, product_height_cm, product_width_cm)
FROM      'C:\olist_products_dataset.csv'
DELIMITER ','
CSV HEADER;

```

#
O intuito do projeto é criar um dashboard no Power BI, para que gestores e analistas de negócio possam ter controle sobre o faturamento anual e mensal da empresa, quantidade de produtos vendidos e valores de cashback entregues.

Abaixo o tratamento realizado em SQL, para criação do DW com as informações referentes a valores de compra e cashback e informações que categorizam as compras:

```
WITH padronizacao_compras as (
SELECT     /* Tratamento para criação de identificador único dos status de compra */
          CASE
              WHEN comp.status_compra LIKE 'approved'    THEN '01.approved'
              WHEN comp.status_compra LIKE 'canceled'    THEN '02.canceled'
              WHEN comp.status_compra LIKE 'created'     THEN '03.created'
              WHEN comp.status_compra LIKE 'delivered'   THEN '04.delivered'
              WHEN comp.status_compra LIKE 'invoiced'    THEN '05.invoiced'
              WHEN comp.status_compra LIKE 'processing'  THEN '06.processing'
              WHEN comp.status_compra LIKE 'shipped'     THEN '07.shipped'	
              WHEN comp.status_compra LIKE 'unavailable' THEN '08.unavailable'
          END as status_compra,
          CAST(comp.dt_compra as DATE) as dt_compra,
/* Tratamento para criação de identificador único dos tipos de pagamento */
          CASE
              WHEN pag.tipo_pagamento LIKE 'boleto'      THEN '01.boleto'
              WHEN pag.tipo_pagamento LIKE 'credit_card' THEN '02.credit card'
              WHEN pag.tipo_pagamento LIKE 'debit_card'  THEN '03.debit card'
              WHEN pag.tipo_pagamento LIKE 'not_defined' THEN '04.not defined'
              WHEN pag.tipo_pagamento LIKE 'voucher'     THEN '05.voucher'
          END as tipo_pagamento,
		  pag.vlr_pagamento,
	  CASE
	      WHEN pag.vlr_pagamento > 0 THEN pag.vlr_pagamento * 0.05
	  END as cashback,
	  itens.preco as preco_unitario_item,
	  itens.quantidade_produtos,
	  itens.frete,
/* Tratamento para criação substiuição de caracteres '_' por '/'*/
	  TRIM(UPPER(REPLACE(prod.produto_categoria, '_', '/'))) as produto_categoria

FROM      compra    as comp
LEFT JOIN (
           SELECT   id_compra,
                    tipo_pagamento,
                    vlr_pagamento

		   FROM     pagamento
          ) as pag
ON        (comp.id_compra = pag.id_compra)
LEFT JOIN (
           SELECT   id_compra,
                    id_produto,
                    preco,
	                frete,
                    COUNT(id_produto) as quantidade_produtos

		   FROM     itens
		   
		   GROUP BY 1,2,3,4
          ) as itens
ON        (comp.id_compra = itens.id_compra)
LEFT JOIN (
           SELECT   id_produto,
                    produto_categoria

		   FROM     produtos

          ) as prod
ON        (prod.id_produto = itens.id_produto)
)

SELECT   /* Separação dos campos de identificação e descrição do status da compra */
         TRIM(SPLIT_PART(padc.status_compra, '.', 1)) as id_status_compra,
         TRIM(SPLIT_PART(padc.status_compra, '.', 2)) as status_compra_desc,
         padc.dt_compra,
         /* Separação dos campos de identificação e descrição do tipo de pagamento */
	 TRIM(SPLIT_PART(padc.tipo_pagamento, '.', 1)) as id_tipo_pagamento,
         TRIM(SPLIT_PART(padc.tipo_pagamento, '.', 2)) as tipo_pagamento_desc,
	 padc.preco_unitario_item as preco_unitario_item,
	 padc.frete,
	 padc.produto_categoria,
	 SUM(padc.cashback)            as vlr_cashback,
	 SUM(padc.vlr_pagamento)       as vlr_compra,
	 SUM(padc.quantidade_produtos) as quantidade_produtos		 

FROM     padronizacao_compras as padc

GROUP BY 1,2,3,4,5,6,7,8
```

Com o uso de ferramentas como dbt, podemos desenvolver modelagens que tratam os dados e os agrupam em DWs. Esse processo auxilia no desenvolvimento de dashboard, assim como em uma melhor visualização dos dados por qualquer área de interesse dentro de uma empresa. Com possibilidade de criação de Data Marts, ou seja, o agrupamento de DWs que tenham o mesmo assunto principal
