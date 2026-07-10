# Analyser: Eficiência e Custo-Benefício na Câmara dos Deputados

Desafio técnico — análise de dados abertos da Câmara dos Deputados, cruzando o uso da
Cota Parlamentar (gastos) com a produtividade legislativa (proposições) de todos os
deputados federais ativos no ano de 2025.

## Objetivo

Ir além de "quem gasta mais" ou "quem propõe mais" e medir **eficiência**: qual a
relação real entre dinheiro público usado e entrega legislativa, no nível individual
de cada parlamentar.

## Fonte de Dados

API de Dados Abertos da Câmara dos Deputados: https://dadosabertos.camara.leg.br/swagger/api.html
- `/deputados` — cadastro dos parlamentares ativos
- `/deputados/{id}/despesas` — despesas da Cota Parlamentar (paginado)
- `/proposicoes` — projetos de lei apresentados, filtrados por autor (paginado)

## Como Rodar

1. Clone o repositório e crie o ambiente:
```bash
   pip install -r requirements.txt
```
2. Abra `analise_camara.ipynb` e execute **Run All**.

O notebook segue uma arquitetura ELT autônoma: na primeira execução, ele consome a API
e salva um cache local em `Dados/*.csv`; execuções seguintes reaproveitam o cache,
sem qualquer necessidade de input manual do usuário.

## Estrutura da Análise

1. **Extração (ELT):** paginação completa por deputado para despesas e proposições.
2. **Transformação (Pandas):** limpeza de tipos, agregação por deputado/categoria/mês/partido,
   e criação da métrica `Custo_por_Projeto`.
3. **Visualização e Storytelling** (4 gráficos, cada um seguido de uma interpretação
   quantitativa impressa logo abaixo):
   - Distribuição da Cota Parlamentar por categoria de despesa
   - Sazonalidade mensal dos gastos em 2025
   - Produtividade legislativa por partido
   - Correlação Gasto x Produtividade por deputado (o "clímax" analítico)
4. **Conclusão geral**, sintetizando as quatro visualizações numa narrativa única.

## Principais Insights

- Os gastos da Cota Parlamentar são fortemente concentrados em poucas categorias de despesa.
- Há sazonalidade clara ao longo do ano, com meses de pico distantes do calendário legislativo regular.
- O ranking de produtividade por partido reflete principalmente o tamanho da bancada — não eficiência.
- Ao corrigir por indivíduo, o notebook identifica deputados "Top Performers" (alta produtividade,
  baixo custo) e casos de "Ineficiência Crítica" (alto custo, baixa entrega), mostrando que gasto
  elevado não garante produtividade legislativa.

## Problemas Encontrados e Soluções

| Problema | Solução aplicada |
|---|---|
| A API pagina os resultados de despesas/proposições (máx. 100 itens por requisição) | Loop de paginação por deputado até retornar página vazia ou incompleta |
| Valores monetários vinham como string em alguns registros | Conversão explícita com `pd.to_numeric(errors='coerce')` para evitar erros de soma |
| Datas de despesas com formatos inconsistentes/nulos | Conversão com `pd.to_datetime(errors='coerce')` e filtro de nulos antes da agregação mensal |
| Deputados sem despesas ou sem proposições no ano geravam `NaN` no merge | `fillna(0)` após o merge, para não quebrar os gráficos e agregações |
| Risco de rate limit da API pública ao rodar ~513 deputados sequencialmente | `time.sleep(0.1)` entre requisições ("pausa educada") + tratamento de exceção por chamada, sem interromper o pipeline inteiro |
| Extração sequencial de ~513 deputados era o principal gargalo do pipeline | Paralelizado com `ThreadPoolExecutor` (8 workers), reduzindo o tempo de extração de ~20min para **5.3min** (~4x mais rápido) |
## Sugestões para Trabalhos Futuros

Detalhadas na última seção do notebook:
1. Paralelização da extração com `ThreadPoolExecutor`.
2. Segmentação de deputados via clustering (KMeans) sobre gasto x produtividade, substituindo o corte fixo pela média.
3. NLP sobre o texto das ementas das proposições, para agrupamento temático mais granular do que a classificação oficial da API.

## Tecnologias

`Python`, `pandas`, `numpy`, `requests`, `matplotlib`, `seaborn`

## Estrutura de Pastas

```
.
├── analise_camara.ipynb
├── requirements.txt
├── README.md
└── Dados/              # cache local gerado na primeira execução (gitignored)
```