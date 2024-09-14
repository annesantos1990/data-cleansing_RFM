# Tratamento dos Dados - Projeto RFM Analysis

Repositório com os detalhes da limpeza de dados do Projeto de Segmentação de Clientes usando a análise RFM

    
→ As etapas de limpeza de dados desse projeto foram feitas no Big Query. 

→ Nome do projeto criado pelo time no Big Query: **hipoteses-proj2**

## Biblioteca de Dados

- Temos três tabelas:
    
    - Tabela de Clientes: O *idcliente* aparece somente uma vez, pois contém a identificação de cada cliente.
    - Tabela de transações: Contém o registro das compras do período. É esperado que haja mais de uma compra para cada cliente.
    - Tabela Resumo/Compra: Resumo de compras contendo o total de compras realizado pelo cliente nesse período para cada categoria de produtos.
    - **Tabela Clientes - 2240 linhas**
    
    
    | **Variável** | **Descrição** |
    | --- | --- |
    | **id_cliente** | Identificador do cliente. Um número inteiro de 1 a 5 dígitos atribuídos exclusivamente a cada cliente. |
    | **ano_nascimento** | Ano de nascimento do cliente. |
    | **nivel_educacao** | Nível de escolaridade do cliente. |
    | **estado_civil** | Estado civil do cliente. |
    | **salarioanualdolar** | Renda anual da família do cliente em dólares. |
    | **criancasatedez_anos** | Número de crianças pequenas até 10 anos na casa do cliente. |
    | **criancasmaisdez_anos** | Número de crianças com mais de 10 anos na casa do cliente. |
    | **data_entrada** | Data de cadastro do cliente na empresa. |
    | **resposta_campanha** | Identifica se o cliente aceitou a campanha de marketing enviada. Onde 1 é sim e 0 é não. |

- **Tabela Transações - 22127 linhas**
    
    
    | **Variável** | **Descrição** |
    | --- | --- |
    | **id_transacao** | Identificador da transação. Um número inteiro de 9 dígitos atribuído exclusivamente a cada transação. |
    | **id_cliente** | Identificador do cliente. Um número inteiro de 1 a 5 dígitos atribuídos exclusivamente a cada cliente. |
    | **data_transacao** | Data de compra no formato ano-mês-dia. |
    | **lugar_transacao** | O local onde a compra foi feita, pode ser online ou na loja. |
- **Tabela Resumo Compra - 2249 linhas**
    
    
    | **Variável** | **Descrição** |
    | --- | --- |
    | **id_cliente** | Identificador do cliente. Um número inteiro de 1 a 5 dígitos atribuídos exclusivamente a cada cliente. |
    | **total_vinho** | Valor monetário gasto em produtos do tipo vinho. |
    | **total_frutas** | Valor monetário gasto em produtos do tipo fruta. |
    | **total_carnes** | Valor monetário gasto em produtos do tipo carne animal. |
    | **total_peixes** | Valor monetário gasto em produtos do tipo pescado. |
    | **total_doces** | Valor monetário gasto em produtos do tipo doce. |
    | **total_outros** | Valor monetário gasto em outros produtos em em geral. |

**Período de Coleta:** 30/07/2020 - 29/06/2022

## Tratamento dos Dados
### Valores nulos

Para identificação dos valores nulos das três tabelas, foi utilizada a seguinte consulta no BigQuery:

```sql
--Contagem dos nulos tabela clientes

SELECT
  COUNT(*)
FROM `projeto-1-segmentacao-rfm.projeto_1.clientes`
WHERE resposta_campanha IS NULL

-- Contagem dos nulos tabela resumo_compra
SELECT
  COUNT(*)
FROM `projeto-1-segmentacao-rfm.projeto_1.resumo_compras`
WHERE total_outros IS NULL

-- Contagem dos nulos tabela transacoes
SELECT
  COUNT(*)
FROM `projeto-1-segmentacao-rfm.projeto_1.transacoes`
WHERE data_transacao IS NULL
```

Foram identificados as seguintes quantidades de valores nulos:

| Clientes |  |
| --- | --- |
| **id_cliente** | 0 |
| **ano_nascimento** | 0 |
| **nivel_educacao** | 0 |
| **estado_civil** | 0 |
| **salario*anual*dolar** | 24 |
| **criancas*ate*dez_anos** | 0 |
| **data_entrada** | 0 |
| **resposta_campanha** | 0 |

| Transações |  |
| --- | --- |
| **id_transacao** | 0 |
| **id_cliente** | 7 |
| **data_transacao** | 0 |
| **lugar_transacao** | 0 |

| Clientes |  |
| --- | --- |
| **id_cliente** | 0 |
| **total_vinho** | 0 |
| **total_frutas** | 0 |
| **total_carnes** | 0 |
| **total_peixes** | 0 |
| **total_doces** | 0 |
| **total_outros** | 0 |

Calculei a média e mediana da variável **salario_*anual_*dolar** para decidir como iria substituir os valores.

```sql
--verificando a média e mediana das variáveis com valores nulos

SELECT
  AVG(salario_anual_dolar) OVER() media_salario,
  PERCENTILE_CONT(salario_anual_dolar, 0.5) OVER() AS mediana_salario
FROM `projeto_1.clientes`
limit 1
```

Resultado:

| Média | 52247.2 |
| --- | --- |
| Mediana | 51381.5 |

👉🏽 Devido aos valores próximos, optei por substituir os valores nulos pela média para não alterar o valor final da média dessa variável.

```sql
--Substituindo os valores nulos pela média
SELECT *,
  COALESCE(salario_anual_dolar, media_salario) AS salario_anual_smedia
FROM `projeto_1.clientes`, (
  SELECT
  AVG(salario_anual_dolar) OVER() media_salario,
  PERCENTILE_CONT(salario_anual_dolar, 0.5) OVER() AS mediana_salario
FROM `projeto_1.clientes`
limit 1
)
```

## Valores Duplicados

Para identificação dos valores nulos das três tabelas, foi utilizada a seguinte consulta no BigQuery:

```sql
SELECT
  COUNT(*) AS cont
FROM `projeto_1.clientes`
GROUP BY id_cliente
HAVING cont >1

SELECT
  COUNT(*) AS cont
FROM `projeto_1.transacoes`
GROUP BY id_transacao
HAVING cont >1

SELECT
  COUNT(*) AS cont
FROM `projeto_1.resumo_compras`
GROUP BY id_cliente
HAVING cont > 1
```

As seguintes colunas foram selecionadas para verificação de valores duplicados:

| Planilha/Aba | **Coluna Analisada** | **Valores duplicados e excluídos** |
| --- | --- | --- |
| Clientes | Id_cliente | 0 |
| Transacoes | Id_transicao | 0 |
| Resumo | Id_cliente | 9 |

Essas colunas foram escolhidas para remover valores duplicados, pois são colunas de ID daquela planilha, conforme a explicação abaixo:

**Planilha Clientes - Coluna Id_cliente:** Esta coluna contém o ID para cada cliente. Ela é distinta da coluna id_cliente na planilha "transacoes", onde a coluna identifica o cliente que realizou a compra. Nessa segunda planilha, podem existir valores duplicados, já que um mesmo cliente pode realizar mais de uma compra no mesmo estabelecimento.

**Planilha Transacoes - Coluna Id_transicao:** Foi escolhida para retirar valores duplicados, pois só pode ter um valor de transição para cada compra.

**Planilha Resumo - Coluna Id_cliente:** Essa é uma planilha de resumo, por isso, tem que possuir somente um valores únicos de id_cliente.

Quando olhamos para as outras colunas da tabela Clientes e verificamos se tem duplicados, vemos que tem 177 valores repetidos como é possível ver na tabela abaixo:

<img src=https://github.com/user-attachments/assets/bc38e6b4-c59d-471d-a09e-7b295223e4ba width="800" height="200" alt="Descrição da imagem">

        
       
