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
