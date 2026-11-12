# Cadeias de Markov e regularidade do consumo de proteína

Projeto final de Álgebra Linear. Pergunta: o consumo de proteína registrado hoje ajuda a prever se a pessoa estará dentro da faixa da sua meta diária amanhã?

Estados do dia: **Abaixo**, **Dentro** ou **Acima** da faixa em torno da meta de proteína. Estimamos uma matriz de transição 3×3 (colunas = estado de origem, cada coluna soma 1), calculamos previsões em vários passos (`P^k`), a distribuição estacionária (autovetor do autovalor 1) e comparamos, em um teste temporal, com a frequência geral dos estados.

## Dados

- **MyFitnessPal Food Diary Dataset** (Kaggle): 587.187 registros diários de 9.896 pessoas, set/2014 a abr/2015. ZIP ≈ 168 MB, TSV ≈ 2,15 GB.
- Link da base no Kaggle: _(preencher com o link "Abrir a base no Kaggle" do roteiro)_
- Origem: Weber e Achananuparp, *Insights from Machine-Learned Diet Success Prediction*.
- A base bruta **não** vai para o Git (está no `.gitignore`).

## Como executar

1. Python 3.10+ e dependências:
   ```bash
   pip install -r requirements.txt
   ```
2. Baixe e descompacte a base e coloque o TSV em `dados/mfp-diaries.tsv`
   (ou aponte para outro caminho com a variável de ambiente `MFP_TSV`, por exemplo `MFP_TSV=/caminho/arquivo.tsv`).
3. Abra e execute **todas as células, em ordem**, de `projeto_markov.ipynb`:
   ```bash
   jupyter notebook projeto_markov.ipynb
   ```
   Para reiniciar o ambiente e rodar tudo pela linha de comando: `jupyter nbconvert --to notebook --execute --inplace projeto_markov.ipynb`.

A leitura é feita em blocos de 50.000 linhas e só interpreta as colunas necessárias; espera-se alguns minutos na base completa.

## Estrutura

| Arquivo | Conteúdo |
|---|---|
| `projeto_markov.ipynb` | Notebook (pandas, NumPy, Matplotlib): preparação → estados → transições → previsões → avaliação → resultados |
| `requirements.txt` | Dependências |
| `resultados/` | Tabelas (CSV) e figuras (PNG) geradas pelo notebook |
| `dados/` | Base bruta (ignorada pelo Git) |

### Saídas em `resultados/`

- `filtros.csv`: registros removidos por cada filtro.
- `resumo_recorte.csv`: pessoas, dias aproveitados e pares de dias consecutivos.
- `contagens_transicao.csv`, `matriz_transicao.csv`: contagens e probabilidades (recorte completo).
- `previsoes_k_passos.csv`: colunas de `P^k` para k = 1, 2, 3, 7, 14, 30.
- `distribuicao_estacionaria.csv`: autovetor do autovalor 1 × frequência geral.
- `matriz_transicao_treino.csv`, `treino_cobertura_origens.csv`: matriz usada na avaliação e se alguma origem caiu na regra de poucos dados.
- `avaliacao.csv`, `avaliacao_diferenca.csv`: métricas com IC 95% e diferença Markov − frequência.
- `fig1_matriz_transicao.png`, `fig2_avaliacao.png`: figuras para o relatório.

## Decisões metodológicas (parâmetros na célula de configuração do notebook)

Os valores abaixo são os **padrões implementados** e devem ser confirmados/ajustados pelo roteiro de definição do modelo antes da avaliação final.

- **Convenção:** `P[i, j]` = P(estado `i` amanhã | estado `j` hoje); `p_amanhã = P @ p_hoje`.
- **Estados:** razão `r = proteína consumida / meta`. Abaixo se `r < 0,90`; Dentro se `0,90 ≤ r ≤ 1,10`; Acima se `r > 1,10` (`BAND_LOW`, `BAND_HIGH`).
- **Filtros** (cada um é contado em `filtros.csv`): data inválida; proteína ou meta ausente; meta fora de 20–400 g; proteína ≤ 0 ou > 500 g; calorias fora de 800–6.000 kcal (registro incompleto; só se as calorias existirem); duplicata (pessoa, data); pessoas com menos de 14 dias válidos.
- **Transições:** só pares de dias consecutivos (diferença de exatamente 1 dia) da **mesma pessoa**; intervalos maiores não viram transição.
- **Recorte:** uma matriz única para o grupo (todas as pessoas que passam nos filtros), por causa da pouca quantidade de transições por pessoa.
- **Estacionária:** autovetor do autovalor mais próximo de 1, normalizado para somar 1; o notebook também verifica regularidade, |λ₂| e a convergência de `P^200`.
- **Avaliação:** corte cronológico (60% do intervalo de datas para treino). Treino = pares com dia de destino ≤ corte; teste = pares com dia de origem > corte (pares que cruzam o corte são descartados). Matriz e frequência de referência são estimadas **só no treino**.
- **Regra para poucos dados:** origem com menos de 30 transições no treino usa a frequência do treino como previsão e é sinalizada.
- **Métricas:** acurácia, erro de Brier e log-loss; IC 95% por bootstrap por pessoa (1000 reamostragens, semente 42).
- Uma meta registrada no app não implica meta validada nutricionalmente; a faixa é uma escolha metodológica, não uma recomendação clínica.

## Observações

- O parser aceita o JSON de `total`/`goal` tanto como dicionário quanto como lista de `{name, value}` e detecta automaticamente as colunas e a presença de cabeçalho. O notebook foi testado com dados sintéticos nesse formato; confira a primeira execução na base real (mensagem "colunas com JSON de total/goal" e a tabela de filtros).
- O relatório (PDF, IEEE de duas colunas) é produzido à parte; este repositório cobre a parte de implementação.
