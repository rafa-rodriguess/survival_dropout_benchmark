# Proposta de Reestruturacao End-to-End (A ate G)

Objetivo: consolidar as celulas de todas as secoes do notebook `dropout_bench_v3.ipynb` em blocos funcionais de ponta a ponta, com execucao independente de memoria e contratos de entrada/saida explicitos.

## Visao Geral

- Plano 1: Fundacao Survival (A)
- Plano 2: Engenharia Temporal e Inputs (B)
- Plano 3: Split, Auditoria e Camada de Preprocessamento (C)
- Plano 4: Modelagem e Benchmark Consolidado (D)
- Plano 5: Auditorias Pos-hoc e Mapa de Dependencias (E)
- Plano 6: Ablacao e Evidencias de Estabilidade (F)
- Plano 7: Explainability e Integracao com Paper (G)

Cada plano deve poder rodar isoladamente, desde que os artefatos de entrada existam no storage padrao (DuckDB + arquivos em `outputs_benchmark_survival`).

---

## Plano 1 - Fundacao Survival (A)

Escopo original:
- A0, A1, A1.1, A1.2, A1.3, A2, A3, A4, A5

Responsabilidade:
- preparar ambiente, paths e metadados de execucao
- registrar fontes OULAD no DuckDB
- auditar integridade estrutural
- construir backbone por enrollment
- definir evento, tempo e censura na tabela survival-ready

Entradas:
- arquivos OULAD em `content/` (ou download habilitado)

Saidas obrigatorias:
- DuckDB: `enrollment_backbone`, `enrollment_survival_ready`
- CSV/Parquet de exportacao dessas tabelas
- auditorias da camada de fundacao em `outputs_benchmark_survival/tables/`
- metadados de ambiente e configuracao

Criterio de pronto:
- `enrollment_survival_ready` existe, com schema validado e auditoria sem erros criticos

---

## Plano 2 - Engenharia Temporal e Inputs (B)

Escopo original:
- B1, B2, B3, B4, B5, B5.1, B5.2, B6, B7, B7.1, B7.2

Responsabilidade:
- expandir base para person-period semanal
- agregar VLE semanal
- enriquecer temporalmente features
- construir tabela enrollment-level para modelagem
- harmonizar configuracoes canonicas de benchmark
- materializar visoes/inputs por familia de modelo
- gerar variantes de janela (sensibilidade)

Entradas:
- `enrollment_survival_ready` (Plano 1)
- configuracao canonica persistida (json)

Saidas obrigatorias:
- DuckDB: `person_period_min`, `vle_weekly_features`, `person_period_enriched`, `enrollment_model_table`
- DuckDB: inputs por familia (`pp_*`, `enrollment_*_input*`)
- tabelas de auditoria e comparacao de janelas

Criterio de pronto:
- todos os inputs de modelagem do benchmark estao materializados e auditados

---

## Plano 3 - Split, Auditoria e Camada de Preprocessamento (C)

Escopo original:
- C1, C2, C2.1, C2.2, C2.3, C2.4, C3

Responsabilidade:
- auditar coerencia final de evento/tempo/censura
- aplicar split temporal principal
- auditar leakage (identidade e contexto modulo/apresentacao)
- materializar particoes splitadas finais
- preparar camada de preprocessamento para modelagem

Entradas:
- tabelas de modelagem e person-period do Plano 2

Saidas obrigatorias:
- `split_assignment` e tabelas derivadas splitadas
- auditorias de leakage e cobertura de split
- tabelas prontas para treino/validacao/teste por familia

Criterio de pronto:
- particoes finais existem e passaram nos checks de leakage e consistencia

---

## Plano 4 - Modelagem e Benchmark Consolidado (D)

Escopo original:
- D1, D1.1, D2, D2.1, D3, D3.1, D4, D4.1, D5, D5.1, D6, D6.1, D6.1.1, D6.2, D6.3

Responsabilidade:
- definir protocolo oficial de avaliacao survival-oriented
- materializar aliases/runtime de benchmark
- treinar e ajustar os quatro modelos-familia
- consolidar comparacao final e leaderboard
- gerar camadas de calibracao expandida para comparabilidade

Entradas:
- particoes finais e tabelas preprocessadas do Plano 3
- configuracao harmonizada de metricas/horizontes

Saidas obrigatorias:
- tabelas de performance por modelo e por horizonte
- leaderboard consolidado do benchmark
- auditorias/tabelas de calibracao expandida
- metadados de protocolo (`benchmark_config.json` e manifestos relacionados)

Criterio de pronto:
- benchmark consolidado reproduzivel com metricas e calibracao exportadas

---

## Plano 5 - Auditorias Pos-hoc e Mapa de Dependencias (E)

Escopo original:
- E0, E1, E2, E3, E4, E5, E6, E7

Responsabilidade:
- documentar e validar dependencias tardias entre blocos
- auditar comparabilidade entre familias
- aprofundar diagnosticos de calibracao por horizonte
- consolidar auditorias de sensibilidade

Entradas:
- tabelas consolidadas de performance/calibracao do Plano 4
- metadados e manifestos de configuracao

Saidas obrigatorias:
- tabelas de auditoria de comparabilidade
- tabelas de slope/intercept de calibracao por horizonte
- tabela unificada de strengthening de calibracao
- tabela unificada de sensibilidade

Criterio de pronto:
- auditorias pos-hoc consolidadas e rastreaveis para uso no paper

---

## Plano 6 - Ablacao e Evidencias de Estabilidade (F)

Escopo original:
- F1, F2, F3, F4, F5, F6, F7, F8

Responsabilidade:
- definir protocolo de ablacao
- materializar variantes e executar treinamento comparativo
- consolidar resultados de ablacao entre familias
- auditar estabilidade (bootstrap) e suposicoes de PH

Entradas:
- dados/modelos/tabelas consolidadas dos Planos 3 e 4
- configuracoes de ablacao e sementes fixas

Saidas obrigatorias:
- resultados de ablacao por familia e consolidados
- auditoria de preprocessamento/tuning consolidada
- auditorias de incerteza bootstrap
- evidencias de PH para arm comparavel

Criterio de pronto:
- evidencias de robustez e estabilidade materializadas com trilha de reproducao

---

## Plano 7 - Explainability e Integracao com Paper (G)

Escopo original:
- G1, G2, G3, G4, G5, G6, G7, G8, G9

Responsabilidade:
- definir protocolo de explainability
- gerar explainability por familia e consolidar narrativa comparativa
- curar/exportar ativos finais para `paper_main` e `paper_appendix`
- validar completude dos ativos TeX e preview final

Entradas:
- artefatos finais de modelagem, auditorias e robustez (Planos 4, 5 e 6)
- contrato TeX e registries de ativos

Saidas obrigatorias:
- tabelas/figuras de explainability por familia e consolidadas
- `paper_tex_asset_registry.csv` e manifestos de suporte
- diretorios curados `paper_main/` e `paper_appendix/`

Criterio de pronto:
- pacote final de evidencia do paper completo, auditado e exportado

---

## Mecanismo para Independencia de Memoria (obrigatorio)

## 1) Contrato de artefatos
- Criar registro unico de artefatos (`artifact_registry`) no DuckDB.
- Cada plano registra:
  - `artifact_name`
  - `producer_plan`
  - `version`
  - `schema_hash`
  - `storage_path`
  - `created_at`
- Cada plano inicia com `require_artifacts(...)` e falha de forma explicita se faltar entrada.

## 2) Configuracao unica e persistida
- Centralizar configuracoes em arquivo (ex.: `outputs_benchmark_survival/metadata/modeling_config.json`).
- Proibir redefinicao manual de parametros criticos em celulas downstream.

## 3) Reidratacao total de estado
- No inicio de cada plano:
  - reabrir conexao DuckDB
  - recarregar configuracao
  - validar presenca de tabelas e schemas esperados
- Nao depender de variaveis globais em memoria entre planos.

## 4) Camadas de harmonizacao
- H1: Harmonizacao Raw (tipos, chaves, nomes)
- H2: Harmonizacao Survival (evento/tempo/censura)
- H3: Harmonizacao de Features (naming e dicionario canonico)
- H4: Harmonizacao de Split (regra unica de particao)
- H5: Harmonizacao de Avaliacao (horizontes e metricas comuns)
- H6: Harmonizacao de Robustez (ablacao, bootstrap, PH)
- H7: Harmonizacao de Publicacao (contrato TeX e naming de ativos finais)

---

## Ordem de Execucao Recomendada

1. Rodar Plano 1 e validar artefatos de fundacao.
2. Rodar Plano 2 e validar inputs de modelagem.
3. Rodar Plano 3 e validar particoes finais e leakage checks.
4. Rodar Plano 4 para treinar/avaliar e consolidar benchmark.
5. Rodar Plano 5 para auditorias pos-hoc de comparabilidade/calibracao.
6. Rodar Plano 6 para ablacao e estabilidade.
7. Rodar Plano 7 para explainability e pacote final do paper.

Se um plano falhar, corrigir no proprio plano e rerodar sem depender do estado de memoria de execucoes anteriores.
