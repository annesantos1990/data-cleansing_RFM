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

<table>
  <tr>
    <td>
      <table>
        <tr><th>Clientes</th><th>Qtd de Nulos</th></tr>
        <tr><td><strong>id_cliente</strong></td><td>0</td></tr>
        <tr><td><strong>ano_nascimento</strong></td><td>0</td></tr>
        <tr><td><strong>nivel_educacao</strong></td><td>0</td></tr>
        <tr><td><strong>estado_civil</strong></td><td>0</td></tr>
        <tr><td><strong>salario*anual*dolar</strong></td><td>24</td></tr>
        <tr><td><strong>criancas*ate*dez_anos</strong></td><td>0</td></tr>
        <tr><td><strong>data_entrada</strong></td><td>0</td></tr>
        <tr><td><strong>resposta_campanha</strong></td><td>0</td></tr>
      </table>
    </td>
    <td>
      <table>
        <tr><th>Transações</th><th>Qtd de Nulos</th></tr>
        <tr><td><strong>id_transacao</strong></td><td>0</td></tr>
        <tr><td><strong>id_cliente</strong></td><td>7</td></tr>
        <tr><td><strong>data_transacao</strong></td><td>0</td></tr>
        <tr><td><strong>lugar_transacao</strong></td><td>0</td></tr>
      </table>
    </td>
    <td>
      <table>
        <tr><th>Clientes</th><th>Qtd de Nulos</th></tr>
        <tr><td><strong>id_cliente</strong></td><td>0</td></tr>
        <tr><td><strong>total_vinho</strong></td><td>0</td></tr>
        <tr><td><strong>total_frutas</strong></td><td>0</td></tr>
        <tr><td><strong>total_carnes</strong></td><td>0</td></tr>
        <tr><td><strong>total_peixes</strong></td><td>0</td></tr>
        <tr><td><strong>total_doces</strong></td><td>0</td></tr>
        <tr><td><strong>total_outros</strong></td><td>0</td></tr>
      </table>
    </td>
  </tr>
</table>

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

Os valores de id_clientes não estão duplicados, mas todas as outras colunas estão.

Código utilizado:

```sql
SELECT
  COUNT(*) AS cont
FROM `projeto_1.clientes`
GROUP BY 
    ano_nascimento,
    nivel_educacao,
    nivel_educacao,
    estado_civil,
    salario_anual_dolar,
    criancas_ate_dez_anos,
    criancas_mais_dez_anos,
    data_entrada,
    resposta_campanha
HAVING cont >1
```

## Unir Tabelas

Os dados foram unidos usando INNER JOIN  para poder unir os dados a partir da coluna id_cliente da tabela *transacoes* que é a tabela que possui menos id_cllientes

No processo de união de tabelas, foram adicionadas algumas Common Tables Express (CTE). Cada uma das CTE com o processo de  tratamento dos dados de cada uma das tabelas originais:

**Limpeza e tratamento de tabela clientes:**

- Nessa CTE foram feitas:
    - Correção dos valores da tabela *nivel_educacao*;
    - Correção dos valores da tabela *estado_civil*;
    - Substituição dos valores nulos da coluna *salario_anual_dolar* pela mediana dos valores dessa variável;
    - E remoção dos duplicados.

```sql
WITH clientes_limpo  AS (
  SELECT *,
  COALESCE(salario_anual_dolar, media_salario) AS salario_anual_smedia,
    CASE
    WHEN nivel_educacao LIKE 'Posgrado' THEN 'Pós-graduação'
    WHEN nivel_educacao LIKE 'Secundaria' THEN 'Ensino Médio'
    WHEN nivel_educacao LIKE 'Grado o superior' THEN 'Nível Superior'
  END AS nivel_educacao_2,

  CASE
    WHEN estado_civil = 'Unión de hecho' THEN 'União Estável'
    WHEN estado_civil = 'Soltero'THEN 'Solteiro'
    WHEN estado_civil = 'Otros' THEN 'Outros'
    WHEN estado_civil = 'Casado' THEN 'Casado'
    WHEN estado_civil = 'Viuda/Viudo'THEN 'Viúva/Viúvo'
    WHEN estado_civil = 'Divorciado' THEN 'Divorciado'
  END AS estado_civil_2
FROM `projeto_1.clientes`, (
  SELECT
  AVG(salario_anual_dolar) OVER() media_salario,
  PERCENTILE_CONT(salario_anual_dolar, 0.5) OVER() AS mediana_salario,
FROM `projeto_1.clientes`
limit 1)
QUALIFY ROW_NUMBER() OVER (PARTITION BY ano_nascimento,nivel_educacao,estado_civil,salario_anual_dolar,criancas_ate_dez_anos,criancas_mais_dez_anos,data_entrada,resposta_campanha) < 2
),

```

**Limpeza e tratamento de tabela resumo_compra:**

- Aqui foi adicionada uma nova variável - *total_compra_cliente* - e retirado os duplicados

```sql
resumo_compra_limpo AS (
  SELECT *,
  total_vinho + total_frutas + total_carnes + total_peixes + total_doces + total_outros AS total_compra_cliente
  FROM `projeto_1.resumo_compras`
  QUALIFY ROW_NUMBER() OVER (PARTITION BY id_cliente) < 2
),
```

**Limpeza e tratamento de tabela transações**:

- Nessa CTE foram feitas:
    - Criação da variável *frequencia*;
    - Criação da variável *data_da_ultima_compra*.

```sql
transicoes_limpo AS (
  SELECT
  id_cliente,
  COUNT(id_transacao) AS frequencia,
  MAX(data_transacao) AS data_da_ultima_compra
  FROM `projeto_1.transacoes`
  GROUP BY id_cliente
)
```

Consulta unificada:

```sql
WITH clientes_limpo  AS (
  SELECT *,
  COALESCE(salario_anual_dolar, media_salario) AS salario_anual_smedia,
    CASE
    WHEN nivel_educacao LIKE 'Posgrado' THEN 'Pós-graduação'
    WHEN nivel_educacao LIKE 'Secundaria' THEN 'Ensino Médio'
    WHEN nivel_educacao LIKE 'Grado o superior' THEN 'Nível Superior'
  END AS nivel_educacao_2,

  CASE
    WHEN estado_civil = 'Unión de hecho' THEN 'União Estável'
    WHEN estado_civil = 'Soltero'THEN 'Solteiro'
    WHEN estado_civil = 'Otros' THEN 'Outros'
    WHEN estado_civil = 'Casado' THEN 'Casado'
    WHEN estado_civil = 'Viuda/Viudo'THEN 'Viúva/Viúvo'
    WHEN estado_civil = 'Divorciado' THEN 'Divorciado'
  END AS estado_civil_2
FROM `projeto_1.clientes`, (
  SELECT
  AVG(salario_anual_dolar) OVER() media_salario,
  PERCENTILE_CONT(salario_anual_dolar, 0.5) OVER() AS mediana_salario,
FROM `projeto_1.clientes`
limit 1)
QUALIFY ROW_NUMBER() OVER (PARTITION BY ano_nascimento,nivel_educacao,estado_civil,salario_anual_dolar,criancas_ate_dez_anos,criancas_mais_dez_anos,data_entrada,resposta_campanha) < 2
),

resumo_compra_limpo AS (
  SELECT *,
  total_vinho + total_frutas + total_carnes + total_peixes + total_doces + total_outros AS total_compra_cliente
  FROM `projeto_1.resumo_compras`
  QUALIFY ROW_NUMBER() OVER (PARTITION BY id_cliente) < 2
),

transicoes_limpo AS (
  SELECT
  id_cliente,
  COUNT(id_transacao) AS frequencia,
  MAX(data_transacao) AS data_da_ultima_compra
  FROM `projeto_1.transacoes`
  GROUP BY id_cliente
)

SELECT 
  transicoes.id_cliente,
  ano_nascimento,
  nivel_educacao_2  AS nivel_educacao,
  estado_civil_2 AS estado_civil,
  salario_anual_smedia AS salario_anual,
  transicoes.lugar_transacao,
  criancas_ate_dez_anos,
  criancas_mais_dez_anos,
  salario_anual_smedia / 12 as salario_mensal,
  2022 - ano_nascimento  AS idade,
  resposta_campanha,
  transicoes.data_da_ultima_compra,
  DATE_DIFF('2022-12-31', transicoes.data_da_ultima_compra,DAY) AS recencia,
  transicoes.frequencia,
  resumo_compra.total_compra_cliente AS valor_monetario,

  

FROM transicoes_aggregated transicoes
JOIN clientes_limpo clientes ON transicoes.id_cliente = clientes.id_cliente
JOIN resumo_compra_limpo resumo_compra  ON  transicoes.id_cliente = resumo_compra.id_cliente
```        
Além das novas variáveis que foram citadas. Também foram criadas as seguintes variáveis:

    - salario_mensal:

    ```
      salario_anual_smedia / 12 as salario_mensal,
    ```

    - Recência
    Que foi melhor detalhado nesse repositório.
