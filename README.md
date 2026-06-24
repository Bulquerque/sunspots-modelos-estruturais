# Modelos estruturais para manchas solares

Analise de series temporais da serie anual de manchas solares (`statsmodels.datasets.sunspots`) usando modelos estruturais, validacao temporal e diagnosticos de residuos.

O notebook principal e [sunspots_modelos_estruturais_ENTREGA_FINAL_aprofundado.ipynb](sunspots_modelos_estruturais_ENTREGA_FINAL_aprofundado.ipynb). Ele foi executado localmente em 2026-06-24 com `jupyter nbconvert --execute` e terminou sem celulas com erro.

## Objetivo

Comparar modelos estruturais para a serie anual de manchas solares, mantendo separacao temporal clara:

- treino de tuning: 1700-1942
- validacao interna: 1943-1975
- validacao final fora da amostra: 1976-2008

A validacao final e usada apenas depois da escolha de hiperparametros e da analise interna.

## Modelos avaliados

- M1: nivel local + 1 harmonico
- M2: nivel local + N harmonicos
- M3: tendencia local + 1 harmonico
- M4: tendencia local + N harmonicos
- M5: Box-Cox + nivel local + ciclo estocastico amortecido
- M6: tendencia + ciclo usual, usado como modelo de conformidade

## Resultado principal

Na validacao final 1976-2008, o melhor desempenho agregado foi do M5:

| modelo | R2_OOS | RMSE | MAE | Bias |
|---|---:|---:|---:|---:|
| M5 Box-Cox + ciclo amortecido | 0.9029 | 17.3470 | 13.2224 | -0.2669 |
| M1 nivel local + H1 | 0.8918 | 18.3086 | 13.8989 | -0.7598 |
| M3 tendencia local + H1 | 0.8911 | 18.3694 | 13.9340 | -1.2606 |
| M6 tendencia + ciclo usual | 0.8448 | 21.9272 | 16.3864 | -1.4741 |

Na ablação, o modelo completo M5 reduziu o RMSE em 1.0224 ponto contra o baseline M3, ganho relativo de 5.57%.

## Leitura critica

A EDA indica ciclo dominante proximo de 11 anos, assimetria e instabilidade de escala. Isso justifica testar tanto harmonicos quanto transformacao Box-Cox e ciclo estocastico amortecido.

Os resultados favorecem o M5 em RMSE, MAE, bias e robustez por janelas finais. Ainda assim, a conclusao deve ser cautelosa: a validacao final tem 33 observacoes, e testes pareados em amostras pequenas podem ser sensiveis. A analise de residuos mostra que o M5 tem Ljung-Box p-value de 0.2188 no teste final, enquanto o M6 ainda deixa autocorrelacao relevante no residuo.

## Artefatos gerados

Os outputs executados estao em [outputs_sunspots_entrega_final](outputs_sunspots_entrega_final):

- tabelas CSV com metricas, diagnosticos, ablação, robustez e parametros estimados
- graficos de EDA, componentes suavizados, residuos e intervalos
- comparacoes de previsao no teste final

Graficos principais:

- [comparacao final dos modelos](outputs_sunspots_entrega_final/14_comparacao_final_principais.png)
- [ablação do modelo vencedor](outputs_sunspots_entrega_final/22_ablacao_modelo_vencedor_rmse.png)
- [intervalo de previsao do M5](outputs_sunspots_entrega_final/23_intervalo_previsao_95_M5.png)
- [periodograma dos residuos](outputs_sunspots_entrega_final/25_periodograma_residuos_modelos_centrais.png)

## Como reproduzir

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace sunspots_modelos_estruturais_ENTREGA_FINAL_aprofundado.ipynb --ExecutePreprocessor.timeout=1200
```

O notebook cria a pasta `outputs_sunspots_entrega_final/` automaticamente.

## Observacao de qualidade

Durante a verificacao foi corrigida uma falha na secao de cobertura dos intervalos: a funcao criava a coluna `covered`, mas o calculo buscava `inside`, o que causava `KeyError`. Depois da correcao, o notebook foi executado por completo.
