# Portfólio de Análise de Dados | Foco na Indústria Farmacêutica e Química

**Rhudson Sampaio | Ciência de Dados & Business Intelligence**

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

### Case 2: Dashboard de Gestão de Carteira e Análise de Churn

* **Área de Impacto:** Comercial e Serviço de Atendimento ao Cliente (SAC).
* **O Desafio:** A equipe comercial necessitava de uma ferramenta analítica para otimizar a gestão da carteira, compreender a saúde da base de clientes e, principalmente, priorizar esforços de reativação comercial de forma estratégica.
* **A Solução:** Desenvolvi o dashboard "Distribuição de Clientes", que segmenta a base em Ativos, Inativos e Perdidos, com base em regras de negócio dinâmicas. O diferencial da solução é um estudo detalhado da frequência de compras dos clientes "Perdidos", permitindo focar a reativação naqueles com maior potencial de retorno.
* **Impacto no Negócio:** A ferramenta capacitou a equipe comercial com dados para maximizar a retenção e a prospecção. A análise de frequência direcionou os esforços de reativação para as oportunidades mais promissoras, otimizando o tempo da equipe e aumentando a taxa de sucesso das campanhas.

<!-- 
![Dashboard de Distribuição de Clientes](images/case-distribuicao-clientes.png) 
-->

---

### Case 3: Dashboard de Análise Financeira e de Risco

* **Área de Impacto:** Financeiro e Logística.
* **O Desafio:** A empresa precisava de uma visão 360° do ciclo financeiro, desde o faturamento até a liquidação, incluindo análises de risco de crédito e identificação de gargalos logísticos que impactavam o faturamento.
* **A Solução:** Criei um dashboard financeiro com quatro pilares estratégicos: 1) Monitoramento de títulos a receber com score de risco por cliente; 2) Análise de Faturado vs. Recebido para avaliar a eficiência do fluxo de caixa; 3) Gestão de saldo de crédito (integrando dados internos e do SERASA); e 4) Acompanhamento de itens com entrega atrasada.
* **Impacto no Negócio:** A solução forneceu uma visão consolidada que otimizou o fluxo de caixa, melhorou a gestão de crédito com base em scores de risco e permitiu a identificação de problemas logísticos antes que afetassem o cliente, protegendo a receita e a satisfação.

<!-- 
![Dashboard de Análise Financeira](images/case-financeiro.png) 
-->

---
