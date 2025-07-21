# Portfólio de Análise de Dados | Foco na Indústria Farmacêutica e Química

**Rhudson Sampaio | Análise de Dados & Business Intelligence**

Graduando em Ciência de Dados pela PUC Minas, com experiência prática no desenvolvimento de dashboards estratégicos no Sankhya ERP. Meu foco é na aplicação de Python, SQL, Power BI e Excel avançado para transformar dados complexos em soluções de negócio mensuráveis.

Este portfólio apresenta uma coleção de dashboards e soluções de BI desenvolvidas para resolver desafios de negócio complexos em diversas áreas, como Comercial, Financeiro e SAC, e gerar impacto mensurável.

---

## 📊 Projetos de Business Intelligence

### Case 1: Solução de BI Gerencial (Dashboard RMG v3)

* **Área de Impacto:** Setor Comercial (Vendas e Gestão).
* **O Desafio:** O setor comercial necessitava de uma ferramenta de Business Intelligence centralizada para acompanhamento 360° da performance de vendas. O objetivo era consolidar análises aprofundadas sobre faturamento, margem de lucro, performance de clientes e curvas ABC, capacitando a equipe com dados para uma atuação mais estratégica.
* **A Solução:** Desenvolvi o Dashboard RMG v3, a principal ferramenta de BI do setor comercial. A solução é composta por mais de 10 módulos interativos que permitem à equipe identificar proativamente riscos, capitalizar oportunidades e avaliar a performance global de forma eficiente.
* **Impacto no Negócio:** A plataforma se tornou a fonte única de verdade para a tomada de decisão da equipe, permitindo ações como a renegociação com clientes de baixa margem (módulo "Grandes Volumes") e a otimização do mix de produtos.

![Menu Principal do Dashboard RMG](images/Menu%20RMG.png)
![Dashboard de Análise de Grandes Volumes](images/Grandes%20Volumes.png)

<details>
<summary>Visualizar Código SQL de Exemplo (Módulo Grandes Volumes)</summary>

```sql
-- Lógica para identificar clientes de alto volume e baixa rentabilidade
-- NOTA: Nomes de tabelas e campos foram genericizados para proteger a confidencialidade.
SELECT
    par.ID_CLIENTE,
    par.NOME_CLIENTE,
    -- Faturamento Líquido (Vendas - Devoluções)
    SUM(
        CASE
            WHEN cab.TIPO_OPERACAO IN ('VENDA_FATURADA', 'VENDA_SIMPLES') THEN cab.VALOR_NOTA
            WHEN cab.TIPO_OPERACAO = 'DEVOLUCAO_VENDA' THEN -cab.VALOR_NOTA
            ELSE 0
        END
    ) AS FATURAMENTO_TOTAL,
    -- Custo Total de Reposição dos Produtos Vendidos
    SUM(ite.QUANTIDADE * NVL(cust.CUSTO_REPOSICAO, 0)) AS CUSTO_TOTAL,
    -- Cálculo da Margem Percentual
    ROUND(
        (
            SUM(
                CASE
                    WHEN cab.TIPO_OPERACAO IN ('VENDA_FATURADA', 'VENDA_SIMPLES') THEN cab.VALOR_NOTA
                    ELSE -cab.VALOR_NOTA
                END
            ) - SUM(ite.QUANTIDADE * NVL(cust.CUSTO_REPOSICAO, 0))
        ) / NULLIF(
            SUM(
                CASE
                    WHEN cab.TIPO_OPERACAO IN ('VENDA_FATURADA', 'VENDA_SIMPLES') THEN cab.VALOR_NOTA
                    ELSE -cab.VALOR_NOTA
                END
            ),
            0
        ) * 100,
        2
    ) AS MARGEM_PERCENTUAL
FROM
    FAT_NOTAS_CABECALHO cab
    JOIN FAT_NOTAS_ITENS ite ON ite.ID_NOTA = cab.ID_NOTA
    JOIN CAD_PARCEIROS par ON par.ID_PARCEIRO = cab.ID_PARCEIRO
    JOIN EST_CUSTOS_PRODUTO cust ON cust.ID_PRODUTO = ite.ID_PRODUTO
WHERE
    cab.DATA_FATURAMENTO BETWEEN :DATA_INICIAL AND :DATA_FINAL
    AND cab.STATUS_NOTA = 'APROVADA'
GROUP BY
    par.ID_CLIENTE,
    par.NOME_CLIENTE
-- Filtro estratégico para encontrar os casos críticos
HAVING
    ROUND(
        (
            SUM(
                CASE
                    WHEN cab.TIPO_OPERACAO IN ('VENDA_FATURADA', 'VENDA_SIMPLES') THEN cab.VALOR_NOTA
                    ELSE -cab.VALOR_NOTA
                END
            ) - SUM(ite.QUANTIDADE * NVL(cust.CUSTO_REPOSICAO, 0))
        ) / NULLIF(
            SUM(
                CASE
                    WHEN cab.TIPO_OPERACAO IN ('VENDA_FATURADA', 'VENDA_SIMPLES') THEN cab.VALOR_NOTA
                    ELSE -cab.VALOR_NOTA
                END
            ),
            0
        ) * 100,
        2
    ) <= 30
ORDER BY
    FATURAMENTO_TOTAL DESC;
```
</details>

---

### Case 2: Dashboard de Performance e Fechamento de Vendas

* **Área de Impacto:** Gestão Comercial, Vendas e Controladoria.
* **O Desafio:** A gestão comercial necessitava de uma ferramenta consolidada para monitorar a performance da força de vendas em tempo real, calcular comissões com base em regras de negócio complexas e acompanhar o atingimento de metas individuais e globais de forma transparente.
* **A Solução:** Desenvolvi o dashboard "Fechamento Vendedoras", uma solução de BI com múltiplos níveis de análise. A plataforma consolida KPIs como Faturamento (Individual e Global), Cesta de Produtos, Positivação e Novos/Reativados. O principal diferencial é o módulo de cálculo de comissões, que aplica regras distintas para clientes de perfil GERAL vs. KA (Key Account).
* **Impacto no Negócio:** A ferramenta automatizou o processo de fechamento de vendas, fornecendo transparência total sobre a performance e o cálculo de comissões. Isso aumentou a motivação da equipe e permitiu que a gestão identificasse rapidamente desvios de meta para ações corretivas, fortalecendo a cultura de dados no setor comercial.

![Menu do Dashboard de Fechamento de Vendedoras](images/Menu%20Fechamento%20Vendedoras.png)
![Tabela de Cálculo de Comissões](images/Comiss%C3%B5es.png)

<details>
<summary>Visualizar Código SQL de Exemplo (Cálculo de Comissão)</summary>

```sql
-- Lógica para cálculo de comissão por perfil de cliente (GERAL vs KA)
-- NOTA: Nomes de tabelas e campos foram genericizados para proteger a confidencialidade.
SELECT
    v.NOME_VENDEDORA AS COLABORADOR,
    p.PERFIL_CLIENTE, -- 'GERAL' ou 'KA'
    SUM(
        CASE
            WHEN c.TIPO_OPERACAO LIKE 'VENDA%' THEN c.VALOR_NOTA
            WHEN c.TIPO_OPERACAO LIKE 'DEVOLUCAO%' THEN -c.VALOR_NOTA
        END
    ) AS FATURAMENTO_LIQUIDO,
    MAX(meta.VALOR_META) AS META,
    -- Lógica de Faixas de Comissão
    CASE
        WHEN p.PERFIL_CLIENTE = 'GERAL' THEN
            CASE
                WHEN (SUM(c.VALOR_NOTA_LIQUIDA) / MAX(meta.VALOR_META)) BETWEEN 1.00 AND 1.09 THEN 0.02 -- 2%
                WHEN (SUM(c.VALOR_NOTA_LIQUIDA) / MAX(meta.VALOR_META)) >= 1.10 THEN 0.03 -- 3%
                ELSE 0
            END
        WHEN p.PERFIL_CLIENTE = 'KA' THEN
            CASE
                WHEN (SUM(c.VALOR_NOTA_LIQUIDA) / MAX(meta.VALOR_META)) BETWEEN 1.00 AND 1.09 THEN 0.015 -- 1.5%
                WHEN (SUM(c.VALOR_NOTA_LIQUIDA) / MAX(meta.VALOR_META)) >= 1.10 THEN 0.025 -- 2.5%
                ELSE 0
            END
    END AS PERCENTUAL_COMISSAO
FROM
    FAT_NOTAS_CABECALHO c
    JOIN CAD_VENDEDORES v ON v.ID_VENDEDOR = c.ID_VENDEDOR
    JOIN CAD_PARCEIROS p ON p.ID_PARCEIRO = c.ID_PARCEIRO
    JOIN PLAN_METAS meta ON meta.ID_VENDEDOR = v.ID_VENDEDOR AND meta.MES_ANO = TO_CHAR(c.DATA_FATURAMENTO, 'YYYY-MM')
WHERE
    c.DATA_FATURAMENTO BETWEEN :DATA_INICIAL AND :DATA_FINAL
    AND c.STATUS_NOTA = 'APROVADA'
GROUP BY
    v.NOME_VENDEDORA,
    p.PERFIL_CLIENTE;
```
</details>

---

### Case 3: Dashboard de Gestão de Carteira e Análise de Churn

* **Área de Impacto:** Comercial e Serviço de Atendimento ao Cliente (SAC).
* **O Desafio:** A equipe comercial necessitava de uma ferramenta analítica para otimizar a gestão da carteira, compreender a saúde da base de clientes e, principalmente, priorizar esforços de reativação comercial de forma estratégica.
* **A Solução:** Desenvolvi o dashboard "Distribuição de Clientes", que segmenta a base em Ativos, Inativos e Perdidos, com base em regras de negócio dinâmicas. O diferencial da solução é um estudo detalhado da frequência de compras dos clientes "Perdidos", permitindo focar a reativação naqueles com maior potencial de retorno.
* **Impacto no Negócio:** A ferramenta capacitou a equipe comercial com dados para maximizar a retenção e a prospecção. A análise de frequência direcionou os esforços de reativação para as oportunidades mais promissoras, otimizando o tempo da equipe e aumentando a taxa de sucesso das campanhas.

![Indicadores de Resumo da Carteira](images/Indicadores%20Carteira.png)
![Gráfico de Potencial de Reativação](images/Clientes%20Perdidos%20por%20Frequ%C3%AAncia.png)

<details>
<summary>Visualizar Código SQL de Exemplo (Classificação de Status)</summary>

```sql
-- Lógica para classificação de status do cliente (Ativo, Inativo, Perdido)
-- NOTA: Nomes de tabelas e campos foram genericizados para proteger a confidencialidade.
SELECT
    PAR.ID_CLIENTE,
    PAR.NOME_CLIENTE,
    ULT.ULTIMA_COMPRA,
    -- Classificação dinâmica com base na data da última compra e no ciclo de vendas atual
    CASE
        WHEN ULT.ULTIMA_COMPRA IS NULL THEN 'PERDIDO'
        WHEN ULT.COMPROU_NO_CICLO = 1 THEN 'ATIVO'
        WHEN ULT.ULTIMA_COMPRA >= ADD_MONTHS(:DATA_INICIAL_CICLO, -4) THEN 'ATIVO'
        WHEN ULT.ULTIMA_COMPRA BETWEEN ADD_MONTHS(:DATA_INICIAL_CICLO, -6) AND ADD_MONTHS(:DATA_INICIAL_CICLO, -4) THEN 'INATIVO'
        ELSE 'PERDIDO'
    END AS STATUS_CLIENTE
FROM
    CAD_PARCEIROS PAR
    LEFT JOIN (
        -- Subconsulta para encontrar a data da última compra de cada cliente
        SELECT
            C.ID_PARCEIRO,
            MAX(C.DATA_FATURAMENTO) AS ULTIMA_COMPRA,
            MAX(CASE WHEN C.DATA_FATURAMENTO >= :DATA_INICIAL_CICLO THEN 1 ELSE 0 END) AS COMPROU_NO_CICLO
        FROM FAT_NOTAS_CABECALHO C
        WHERE C.TIPO_OPERACAO LIKE 'VENDA%' AND C.STATUS_NOTA = 'APROVADA'
        GROUP BY C.ID_PARCEIRO
    ) ULT ON ULT.ID_PARCEIRO = PAR.ID_PARCEIRO
WHERE
    PAR.TIPO_CLIENTE = 'ATIVO';
```
</details>

---

### Case 4: Dashboard de Metas e Performance Individual

* **Área de Impacto:** Força de Vendas e Gestão Comercial.
* **O Desafio:** Fornecer a cada vendedora uma visão clara, detalhada e acionável de sua performance individual em relação a um conjunto de metas personalizadas, além de oferecer uma visão de projeção de vendas para o fechamento do ciclo.
* **A Solução:** Criei um dashboard de performance individual que consolida os KPIs mais importantes para cada vendedora: Faturamento, Positivação, Cesta de Produtos e Clientes Reativados. A solução inclui um velocímetro (gauge chart) para o acompanhamento da meta de faturamento e um botão de navegação para um nível de detalhe com a projeção de vendas futuras.
* **Impacto no Negócio:** O dashboard empoderou as vendedoras com dados em tempo real sobre sua performance, aumentando o engajamento e a responsabilidade sobre as metas. A visão de projeção permitiu uma atuação mais proativa para garantir o atingimento dos objetivos ao final de cada ciclo de vendas.

![Dashboard de Metas Individuais](images/Metas%20Individuais%20principal.png)
![Dashboard de Projeção de Vendas](images/Proje%C3%A7%C3%A3o.png)

<details>
<summary>Visualizar Código SQL de Exemplo (KPI de Positivação)</summary>

```sql
-- Lógica para cálculo do KPI de Positivação
-- NOTA: Nomes de tabelas e campos foram genericizados para proteger a confidencialidade.
SELECT
    TRUNC(
        (
            -- Clientes que compraram no ciclo atual
            (SELECT COUNT(DISTINCT c.ID_PARCEIRO)
             FROM VENDAS_CABECALHO c
             WHERE c.ID_VENDEDOR = :ID_VENDEDOR
               AND TRUNC(c.DATA_FATURAMENTO) BETWEEN :INICIO_CICLO AND :FIM_CICLO)
            /
            -- Total de clientes ativos nos 4 meses anteriores ao ciclo
            NULLIF(
                (SELECT COUNT(DISTINCT c2.ID_PARCEIRO)
                 FROM VENDAS_CABECALHO c2
                 WHERE c2.ID_VENDEDOR = :ID_VENDEDOR
                   AND TRUNC(c2.DATA_FATURAMENTO) BETWEEN ADD_MONTHS(:INICIO_CICLO, -4) AND (:INICIO_CICLO - 1)),
                0
            )
        ) * 100,
        2
    ) AS PERCENTUAL_POSITIVACAO
FROM DUAL;
```
</details>

---

