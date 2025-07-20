# Portfólio de Análise de Dados | Foco na Indústria Farmacêutica e Química

**Rhudson Sampaio | Análise de Dados & Business Intelligence**

Profissional com experiência na aplicação de SQL, ETL e visualização de dados para gerar soluções estratégicas no Sankhya ERP, com especialização na indústria farmacêutica e química.

Este portfólio apresenta um estudo de caso aprofundado da **Solução de BI RMG v3**, uma plataforma de análise de dados desenvolvida para resolver desafios de negócio complexos e gerar impacto mensurável.

---

## 📊 Estudo de Caso: Solução de BI RMG v3

### 1. O Desafio de Negócio e a Visão da Solução

> O Dashboard **RMG v3** foi concebido para ser a principal ferramenta de acompanhamento gerencial da empresa. A necessidade era centralizar análises aprofundadas sobre **faturamento**, **margem de lucro**, **performance de clientes** e **curvas ABC**, permitindo à equipe de gestão e comercial identificar proativamente riscos, capitalizar oportunidades e avaliar a performance global de forma estratégica e eficiente.

Para atender a essa demanda, projetei uma plataforma centralizada com múltiplos módulos de análise, acessíveis através de um menu principal intuitivo.

![Menu Principal do Dashboard RMG](images/Menu%20RMG.png)

---

### 2. Deep Dive: Módulo de Análise de Rentabilidade ("Grandes Volumes")

Um dos desafios mais críticos identificados foi a necessidade de gerenciar clientes que, apesar de gerarem um alto faturamento, apresentavam margens de lucro abaixo do ideal, corroendo a rentabilidade geral da empresa.

* **O Desafio (Situação):** A diretoria suspeitava que a rentabilidade de alguns clientes de alto volume não era saudável, mas não havia uma ferramenta para quantificar esse problema e identificar os casos específicos.

* **A Missão (Tarefa):** Minha responsabilidade foi criar um módulo específico dentro da solução RMG para analisar o cruzamento entre faturamento e margem de lucro, definindo como "Grandes Volumes" os clientes com alto faturamento e margem de lucro abaixo de 30%.

* **A Solução (Ação):** Desenvolvi um dashboard detalhado que exibe uma lista filtrada desses clientes, seus respectivos faturamentos, custos associados e a margem de lucro calculada. A tela permite que a gestão visualize instantaneamente os pontos de atenção e tome decisões informadas.

![Dashboard de Análise de Grandes Volumes](images/Grandes%20Volumes.png)

---

### 3. A Prova Técnica: A Lógica por Trás dos Dados (SQL)

A inteligência deste módulo reside na sua consulta SQL. Para criar a análise, foi necessário desenvolver uma query complexa que unisse múltiplas tabelas do ERP para calcular o faturamento líquido e o custo de reposição de cada venda, agregando os resultados por cliente.

A cláusula `HAVING` foi o elemento chave para filtrar e exibir apenas os clientes que se enquadravam na regra de negócio (margem <= 30% e alto faturamento), entregando a informação precisa que a gestão necessitava.

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

---

### 4. Resultados e Impacto no Negócio

* **Tomada de Decisão Baseada em Dados:** A ferramenta eliminou a "percepção" e forneceu dados concretos, permitindo que a diretoria e a equipe comercial focassem nos clientes que realmente necessitavam de uma reavaliação estratégica.
* **Ação Comercial Direcionada:** Com os insights do dashboard, a equipe comercial renegociou acordos, ajustou o mix de produtos e otimizou os preços para os clientes de baixa margem.
* **Impacto Financeiro:** As ações tomadas resultaram em um **aumento médio de 8% na margem de lucro** dos clientes analisados no semestre seguinte, otimizando a rentabilidade da empresa sem comprometer o volume de faturamento.
