# Monitoramento da Fauna — Pipeline de Dados no Databricks

Checkpoint da trilha Fast Track IA. Pipeline de engenharia de dados no
Databricks (Free Edition, Unity Catalog) que transforma os registros brutos
de 12 câmeras automáticas de um parque nacional de Cerrado em evidências
para 5 perguntas de pesquisa + 3 investigações bônus.

Dataset base: [LucasKiraly/fast-track-ia-t1](https://github.com/LucasKiraly/fast-track-ia-t1)
(5.000 registros, 30 dias de monitoramento, 13 espécies em 3 grupos faunísticos).

## Arquitetura

Arquitetura medallion, com cada camada persistida como tabela Delta no
catálogo `fauna.monitoramento`:
## Notebooks

| Notebook | Conteúdo |
|---|---|
| `00_setup_ambiente` | Catálogo, schema e volume; upload do dataset |
| `01_ingestao_bronze` | Leitura do JSON bruto → `bronze_registros` |
| `02_tratamento_silver` | Tipos, qualidade de dados → `silver_registros` |
| `03_enriquecimento_gold` | `dim_camera`, período do dia → `gold_registros` |
| `04_analise_desafios` | Desafios 1 a 5 |
| `05_desafios_bonus` | Bônus 2, 3 e 4 (Bônus 1 é respondido na Fase 2) |

## Decisões de qualidade de dados (Bônus 1)

Documentadas em detalhe no notebook `02_tratamento_silver`. Resumo: nenhum
registro é descartado no pipeline. `temperatura_c` (4,94%) e `umidade_pct`
(3,50%) têm ausências reais de sensor, mantidas como nulo. Um grupo de 10
registros sinalizados como "duplicados" foi inspecionado manualmente e
confirmado como eventos distintos coincidindo no mesmo minuto (limitação de
granularidade do timestamp, não erro de dado).

## Principais achados

- **Capivara** é fortemente gregária (99,2% dos registros em grupo) e diurna
  (78,6% entre manhã e tarde).
- **Lobo-guará** é solitário (74,8%), crepuscular (picos ao amanhecer e
  anoitecer) e concentrado no Cerrado aberto (59,5%).
- **Onça-pintada** é noturna/crepuscular (80% à noite/madrugada) e
  concentrada em Mata de galeria (85%) — amostra pequena (20 registros),
  recomendação tratada como indicativo, não certeza estatística.
- **Anfíbios** aparecem quase 2x mais em umidade alta (>80%) do que os
  demais grupos (65,8% vs 33,9%) — vale monitorar em janelas úmidas.
- **Veredas e áreas úmidas** concentra a maior atividade geral (1.817
  registros, média de 454,3/câmera) entre as três áreas do parque.
- **Duração do registro não se correlaciona** com espécie, grupo taxonômico
  ou tamanho do grupo (correlação ≈ 0) — provavelmente reflete o sensor de
  movimento, não comportamento biológico.

## Considerações de produção (não implementadas neste checkpoint)

- **Modelo fato/dimensão:** `gold_registros` já funciona como uma fato
  "achatada" + `dim_camera`. Formalizar com `dim_especie` e `dim_tempo`
  facilitaria conectar ferramentas de BI (Databricks SQL Dashboard, Power BI).
- **Tempo real:** tecnicamente viável via Auto Loader/Structured Streaming,
  mas não se justifica para este cenário — a pergunta de pesquisa não muda
  segundo a segundo. Faria sentido apenas para um caso de uso diferente:
  alerta operacional imediato (ex.: detecção de onça-pintada para equipe de
  campo, ou detecção de possível caça ilegal).