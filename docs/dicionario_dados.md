# Dicionário de dados — Olist

## Fonte e escopo

- **Dataset:** [Brazilian E-Commerce Public Dataset by Olist (Kaggle)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce).
- **Origem:** nove arquivos CSV mantidos sem modificações em `data/raw/`.
- **Estado:** reconhecimento inicial (BK-001). Tipos e regras de negócio definitivas serão tratados em etapa posterior.
- **Granularidade:** cada tabela descreve uma unidade diferente. Consultas entre tabelas devem respeitar essas unidades para evitar contagem dupla.

## Tabelas e chaves

| Tabela CSV | Linhas | Granularidade | Chave candidata / observação |
|---|---:|---|---|
| `olist_customers_dataset.csv` | 99.441 | Registro de cliente associado a pedido | `customer_id` |
| `olist_orders_dataset.csv` | 99.441 | Pedido | `order_id` |
| `olist_order_items_dataset.csv` | 112.650 | Item de pedido | (`order_id`, `order_item_id`) |
| `olist_order_payments_dataset.csv` | 103.886 | Registro de pagamento de pedido | (`order_id`, `payment_sequential`) |
| `olist_order_reviews_dataset.csv` | 99.224 | Registro de avaliação de pedido | (`review_id`, `order_id`) no dataset analisado |
| `olist_products_dataset.csv` | 32.951 | Produto cadastrado | `product_id` |
| `olist_sellers_dataset.csv` | 3.095 | Vendedor cadastrado | `seller_id` |
| `olist_geolocation_dataset.csv` | 1.000.163 | Registro geográfico | Sem chave única identificada; prefixo CEP se repete |
| `product_category_name_translation.csv` | 71 | Tradução de categoria | `product_category_name` |

*As chaves acima foram inferidas por verificações de unicidade/nulos no conjunto de dados; os CSVs não impõem constraints de banco de dados.*

## Campos por tabela

### `customers`

| Campo | Significado |
|---|---|
| `customer_id` | Identificador de registro de cliente usado na ligação com `orders` |
| `customer_unique_id` | Identificador do comprador entre pedidos diferentes; adequado para contar compradores distintos |
| `customer_zip_code_prefix` | Prefixo de CEP do cliente (identificador geográfico; representar como texto de 5 posições) |
| `customer_city` | Cidade do cliente |
| `customer_state` | UF do cliente |

**Importante:** `customer_id` é exclusivo por pedido no conjunto analisado; o mesmo `customer_unique_id` pode aparecer em vários registros de `customers`.

### `orders`

| Campo | Significado |
|---|---|
| `order_id` | Identificador único do pedido |
| `customer_id` | Referência a `customers.customer_id` |
| `order_status` | Estado do pedido (`delivered`, `shipped`, `canceled`, etc.) |
| `order_purchase_timestamp` | Data/hora de realização do pedido |
| `order_approved_at` | Data/hora de aprovação do pedido |
| `order_delivered_carrier_date` | Data/hora registrada de envio à transportadora |
| `order_delivered_customer_date` | Data/hora registrada de entrega ao cliente |
| `order_estimated_delivery_date` | Data prevista de entrega |

As datas foram carregadas inicialmente como texto e precisam de conversão para data/hora antes de análises temporais.

### `order_items`

| Campo | Significado |
|---|---|
| `order_id` | Referência ao pedido |
| `order_item_id` | Número sequencial do item dentro do pedido (não é globalmente único) |
| `product_id` | Referência ao produto |
| `seller_id` | Referência ao vendedor |
| `shipping_limit_date` | Prazo de envio do item (carregado como texto) |
| `price` | Preço registrado do item |
| `freight_value` | Valor de frete registrado para o item |

### `order_payments`

| Campo | Significado |
|---|---|
| `order_id` | Referência ao pedido |
| `payment_sequential` | Sequência do registro de pagamento dentro do pedido |
| `payment_type` | Tipo ou meio de pagamento |
| `payment_installments` | Quantidade de parcelas informada |
| `payment_value` | Valor do registro de pagamento |

Um pedido pode ter mais de um registro de pagamento.

### `order_reviews`

| Campo | Significado |
|---|---|
| `review_id` | Identificador da avaliação, **não único isoladamente** neste dataset |
| `order_id` | Referência ao pedido; também pode aparecer em várias avaliações |
| `review_score` | Nota atribuída na avaliação |
| `review_comment_title` | Título do comentário, quando informado |
| `review_comment_message` | Texto do comentário, quando informado |
| `review_creation_date` | Data da avaliação (carregada como texto) |
| `review_answer_timestamp` | Data/hora da resposta ou registro da avaliação (carregada como texto) |

A combinação (`review_id`, `order_id`) não apresenta duplicatas nem nulos no conjunto observado. A regra para agregação de notas por pedido ainda será definida.

### `products`

| Campo | Significado |
|---|---|
| `product_id` | Identificador único do produto |
| `product_category_name` | Nome original da categoria |
| `product_name_lenght` | Comprimento do nome informado (grafia original `lenght`) |
| `product_description_lenght` | Comprimento da descrição (grafia original `lenght`) |
| `product_photos_qty` | Quantidade de fotos cadastradas |
| `product_weight_g` | Peso do produto em gramas |
| `product_length_cm` | Comprimento em centímetros |
| `product_height_cm` | Altura em centímetros |
| `product_width_cm` | Largura em centímetros |

### `sellers`

| Campo | Significado |
|---|---|
| `seller_id` | Identificador único do vendedor |
| `seller_zip_code_prefix` | Prefixo de CEP do vendedor (tratar como texto de 5 posições) |
| `seller_city` | Cidade do vendedor |
| `seller_state` | UF do vendedor |

### `geolocation`

| Campo | Significado |
|---|---|
| `geolocation_zip_code_prefix` | Prefixo de CEP, repetido em vários registros |
| `geolocation_lat` | Latitude |
| `geolocation_lng` | Longitude |
| `geolocation_city` | Cidade registrada |
| `geolocation_state` | UF registrada |

**Cuidado:** não unir diretamente esta tabela a clientes/vendedores pelo prefixo de CEP sem antes definir uma linha geográfica representativa por prefixo. O prefixo não é chave única.

### `product_category_name_translation`

| Campo | Significado |
|---|---|
| `product_category_name` | Categoria original, relacionada a `products.product_category_name` |
| `product_category_name_english` | Tradução para inglês |

## Relações confirmadas

| Origem → destino | Campo de ligação | Observação |
|---|---|---|
| `orders` → `customers` | `customer_id` | Nenhuma referência inválida; `customer_id` é único em ambas as tabelas do dataset |
| `order_items` → `orders` | `order_id` | 0 itens órfãos; 775 pedidos sem itens |
| `order_items` → `products` | `product_id` | 0 produtos inexistentes; todos os produtos aparecem em itens |
| `order_items` → `sellers` | `seller_id` | 0 vendedores inexistentes; todos aparecem em itens |
| `order_payments` → `orders` | `order_id` | 0 pagamentos órfãos; 1 pedido sem pagamento |
| `order_reviews` → `orders` | `order_id` | 0 avaliações órfãs; alguns pedidos possuem várias avaliações |
| `products` → `product_category_name_translation` | `product_category_name` | 2 categorias utilizadas sem tradução; 610 produtos sem categoria |
| `customers`/`sellers` ↔ `geolocation` | Prefixo CEP | Relação de consulta geográfica, **não** FK única; 157/7 prefixos distintos sem cobertura |

## Cuidados para as métricas futuras

- **Clientes distintos:** contar `customer_unique_id`, não `customer_id`.
- **Pedidos:** contar `order_id` e definir quais status são elegíveis para cada indicador.
- **Receita:** distinguir `price`, `freight_value` e `payment_value` (conceitos diferentes); definir a regra de negócio antes de somar.
- **JOINs de granularidades distintas:** agregar pagamentos e itens por pedido antes de reuni-los, evitando multiplicação de linhas.
- **Avaliação média:** decidir se a unidade é registro de avaliação ou pedido.
- **Localização:** tratar prefixo CEP como texto; garantir unicidade na dimensão geográfica antes de JOIN.

Ver também [`qualidade_dados.md`](qualidade_dados.md) e [`modelo_relacional.md`](modelo_relacional.md).
