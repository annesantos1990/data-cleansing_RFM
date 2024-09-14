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
        
        **id_cliente:** Identificador do cliente. Um número inteiro de 1 a 5 dígitos atribuídos exclusivamente a cada cliente.
        
        **ano_nascimento:** Ano de nascimento do cliente.
        
        **nivel_educacao:** Nível de escolaridade do cliente.
        
        **estado_civil:** Estado civil do cliente.
        
        **salario*anual*dolar:** Renda anual da família do cliente em dólares.
        
        **criancas*ate*dez_anos:** Número de crianças pequenas até 10 anos na casa do cliente.
        
        **criancas*mais*dez_anos:** Número de crianças com mais de 10 anos na casa do cliente.
        
        **data_entrada:** Data de cadastro do cliente na empresa.
        
        **resposta_campanha:** Identifica se o cliente aceitou a campanha de marketing enviada. Onde 1 é sim e 0 é não.
        
    
    - **Tabela Transações - 22127 linhas**
        
        **id_transacao:** Identificador da transação. Um número inteiro de 9 dígitos atribuído exclusivamente a cada transação.
        
        **id_cliente:** Identificador do cliente. Um número inteiro de 1 a 5 dígitos atribuídos exclusivamente a cada cliente.
        
        **data_transacao:** Data de compra no formato ano-mês-dia.
        
        **lugar_transacao:** O local onde a compra foi feita, pode ser online ou na loja.
        
    
    - **Tabela Resumo Compra - 2249 linhas**
        
        **id_cliente:** Identificador do cliente. Um número inteiro de 1 a 5 dígitos atribuídos exclusivamente a cada cliente.
        
        **total_vinho:** Valor monetário gasto em produtos do tipo vinho.
        
        **total_frutas:** Valor monetário gasto em produtos do tipo fruta.
        
        **total_carnes:** Valor monetário gasto em produtos do tipo carne
        
        animal.
        
        **total_peixes:** Valor monetário gasto em produtos do tipo pescado.
        
        **total_doces:** Valor monetário gasto em produtos do tipo doce.
        
        **total_outros:** Valor monetário gasto em outros produtos em em geral.
