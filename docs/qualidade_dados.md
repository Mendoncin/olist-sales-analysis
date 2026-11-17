# Qualidade dos dados — Olist (BK-001)

Este arquivo registra **observações** da exploração inicial. **Nenhuma correção foi aplicada** aos arquivos em `data/raw/`. Tratamentos e critérios para indicadores serão decididos no próximo backlog.

| Código | Observação | Evidência | Impacto / encaminhamento |
|---|---|---|---|
| **DQ-001** | Pedidos marcados como entregues sem data de entrega | 8 pedidos `delivered` com `order_delivered_customer_date` nulo; 7 têm data de envio à transportadora | Não permitem calcular prazo real de entrega. Preservar em métricas comerciais elegíveis; excluir de indicadores que exigem data de entrega, explicitando a regra. |
| **DQ-002** | Pedidos sem itens associados | 775 pedidos: 603 `unavailable`, 164 `canceled`, 5 `created`, 2 `invoiced`, 1 `shipped` | Investigar critérios de pedido válido; não usar todos os pedidos como denominador sem definir a métrica. |
| **DQ-003** | Pedido entregue sem registro de pagamento | 1 pedido `delivered`, ID `bfbd0f9bdef84302105ad712db648a6c`, sem linhas em `order_payments` | Divergência entre status e registros disponíveis; não inferir falta de pagamento real. |
| **DQ-004** | Múltiplos registros e identificadores de avaliações repetidos | 99.224 linhas; `review_id` com 814 repetições adicionais, `order_id` com 551; 0 linhas integralmente duplicadas; chave (`review_id`, `order_id`) única e sem nulos | Não deduplicar por `review_id` ou por `order_id` sem política de agregação; média por avaliação difere de média por pedido. |
| **DQ-005** | Categorias sem tradução | 73 categorias usadas em produtos e 71 traduzidas; ausentes `pc_gamer` e `portateis_cozinha_e_preparadores_de_alimentos` | Preservar categoria original; fornecer rótulo alternativo no tratamento. |
| **DQ-006** | Prefixos de CEP sem correspondência geográfica | 157 dos 14.994 prefixos distintos de clientes; 7 dos 2.246 de vendedores, ausentes de `geolocation` | Preservar registros em JOINs; ausência de coordenada não equivale a ausência de cidade/UF. Percentuais são de **prefixos**, não de pessoas. |
| **DQ-007** | Prefixos geográficos repetidos | 1.000.163 linhas para 19.015 prefixos; 981.148 ocorrências além da primeira | JOIN direto por prefixo multiplica linhas. Criar representação geográfica por prefixo antes de usar em análises. |
| **DQ-008** | Datas importadas como texto e prefixos CEP como inteiros | Colunas de datas aparecem como `str` no Pandas; prefixos de CEP como `int64` | Converter datas para `datetime`; tratar prefixos como textos de 5 caracteres para preservar zeros iniciais. |
| **DQ-009** | Campos opcionais e atributos ausentes | `order_reviews`: 87.656 títulos e 58.247 comentários ausentes. `products`: 610 ausências em categoria/nome/descrição/fotos; 2 em peso e cada dimensão | Nulos de comentários podem ser legítimos. Não excluir avaliações apenas pela falta de texto; tratar atributos conforme uso. |
| **DQ-010** | Datas de `orders` ausentes | `order_approved_at`: 160; `order_delivered_carrier_date`: 1.783; `order_delivered_customer_date`: 2.965 | Relacionar ausência ao status antes de imputar ou eliminar. Entre os sem data de entrega: 1.107 `shipped`, 619 `canceled`, 609 `unavailable`, 314 `invoiced`, 301 `processing`, 8 `delivered`, 5 `created`, 2 `approved`. |

## Verificações que passaram

- `orders.order_id`, `customers.customer_id`, `products.product_id` e `sellers.seller_id`: sem duplicatas nem nulos.
- `order_items`: (`order_id`, `order_item_id`) única e não nula.
- `order_payments`: (`order_id`, `payment_sequential`) única e não nula.
- `order_reviews`: (`review_id`, `order_id`) única e não nula.
- Referências de `orders` a `customers`, de `order_items` a `orders`/`products`/`sellers`, de `order_payments` a `orders` e de `order_reviews` a `orders`: **0 referências inexistentes**.
- `product_category_name_translation.product_category_name`: 71 valores únicos, sem nulos.

## Decisões pendentes para o próximo backlog

1. Definir pedidos válidos para receita, quantidade de pedidos, ticket médio e satisfação.
2. Definir receita de produtos vs. frete vs. pagamentos, respeitando a granularidade.
3. Definir a política de múltiplas avaliações para métricas por pedido.
4. Estabelecer a dimensão de prefixos geográficos, sem JOIN duplicador.
5. Converter datas, padronizar prefixos CEP e dar nomes alternativos às categorias sem tradução.

*O objetivo é manter rastreabilidade do que foi observado e separar evidência de hipótese; este documento não é uma auditoria completa.*
