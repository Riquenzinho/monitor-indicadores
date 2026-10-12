# Monitor de Indicadores do Varejo — Pipeline de Dados

## Sobre este projeto

Pipeline que coleta séries diárias da API pública do Banco Central (SGS),
processa em camadas (bronze/silver/gold) e orquestra com Airflow.

**História de negócio:** acompanhar os indicadores que pressionam custo e
consumo no varejo — dólar (custo de produto importado), Selic (crédito e
conversão) e IPCA (inflação vs. reajuste de preço).

**O objetivo é APRENDER engenharia de dados.** Sou Analista de Dados buscando
vaga de Engenheiro de Dados Júnior e preciso saber explicar cada parte deste
projeto numa entrevista técnica. Um repositório que eu não sei defender é pior
do que nenhum.

## A fonte

API SGS do Banco Central — pública, sem chave, sem cadastro.

```
https://api.bcb.gov.br/dados/serie/bcdata.sgs.{codigo}/dados
    ?formato=json&dataInicial=DD/MM/AAAA&dataFinal=DD/MM/AAAA
```

Retorno: lista de `{"data": "01/09/2026", "valor": "5.1570"}`.

Séries sugeridas (confirmar os códigos no catálogo do SGS antes de usar):

- **1** — Dólar comercial, venda (PTAX), diária
- **432** — Taxa Selic meta, diária
- **433** — IPCA, variação mensal

Escolher de 3 a 5 séries. Misturar diárias e mensais de propósito: obriga a
pensar granularidade na camada gold.

**Por que esta fonte:** a API aceita intervalo de datas. Isso torna o backfill
do Airflow real — a DAG rodando para uma data passada busca de fato o dado
daquele dia. Com uma API que só devolve o "agora", backfill vira encenação.

## Como me ajudar

- Explique as decisões ANTES de escrever código
- Sempre diga o PORQUÊ, não só o COMO
- Quando o item estiver na lista "eu escrevo", me oriente em vez de resolver
- Pode escrever direto o que está na lista "pode fazer"
- Se eu pedir para pular etapa por pressa, me lembre do objetivo acima

## Eu escrevo (oriente, não resolva)

- Lógica das transformações bronze → silver → gold
- Regras de qualidade de dados e o que fazer quando falham
- Garantia de idempotência
- Corpo das tasks da DAG e a ordem entre elas

## Pode fazer direto

- Docker e docker-compose do Airflow
- Estrutura de pastas e .gitignore
- Boilerplate: imports, configuração de log, leitura de .env
- Comandos de Git quando eu travar
- README final

## Arquitetura

**bronze** — resposta crua da API em Parquet, particionada por data de
execução e por série. Sem transformação. Se a API mudar o formato, o original
continua aqui.

**silver** — dado limpo e tipado: serie_codigo, serie_nome, data_ref (date),
valor (decimal), data_ingestao (timestamp). Deduplicado pela chave
serie_codigo + data_ref.

**gold** — tabelas de análise: séries lado a lado por data, variação diária e
acumulada, e uma visão mensal consolidada.

**Airflow** — DAG diária, tasks em sequência (ingestão → silver → qualidade →
gold), com retry, backoff e idempotência.

## Qualidade — regras antes de promover silver → gold

- Nenhum valor nulo onde deveria haver valor
- Nenhuma chave serie_codigo + data_ref duplicada
- Valor dentro de faixa plausível para a série (dólar entre 1 e 20, por
  exemplo) — pega erro de parsing silencioso
- Volume de linhas dentro do esperado para o período

**Regra que exige pensar:** a série não tem valor em fim de semana nem em
feriado. "Veio vazio" nem sempre é falha. A validação precisa distinguir
ausência esperada de falha de ingestão — e essa distinção é o ponto mais
interessante do projeto para falar em entrevista.

Se a qualidade falhar, a DAG para e avisa. Não publica dado ruim.

## Estrutura

```
monitor-indicadores/
├── src/
│   ├── ingestao.py
│   ├── transformacao.py
│   ├── qualidade.py
│   └── config.py
├── dags/
│   └── monitor_indicadores_dag.py
├── dados/
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── tests/
├── docker-compose.yml
├── .env
├── .gitignore
└── README.md
```

## Módulos

Cada módulo segue o mesmo ciclo: explicação do conceito → esqueleto pronto
(imports, log, assinaturas) → eu escrevo só o miolo da lógica → revisão → eu
explico com minhas palavras → commit e push.

- [x] **0 — Ambiente**: Git, .gitignore, estrutura de pastas
- [ ] **1 — Ingestão**: chamada à API com parâmetro de data, timeout,
  tratamento de erro e retry
- [ ] **2 — Bronze**: resposta crua em Parquet particionado por data de
  execução e por série
- [ ] **3 — Silver**: tipagem (data em date, valor em decimal), deduplicação
- [ ] **4 — Airflow**: Docker, primeira DAG, tasks em sequência
- [ ] **5 — Robustez**: retry com backoff, idempotência, backfill de 2 anos
- [ ] **6 — Gold**: séries lado a lado, variação diária e acumulada, visão mensal
- [ ] **7 — Qualidade**: regras entre silver e gold, incluindo dia sem pregão;
  a DAG para e avisa
- [ ] **8 — Teste**: um teste com pytest
- [ ] **9 — Dashboard**: leitura da camada gold
- [ ] **10 — Entrega**: README com diagrama, limpeza, simulado de perguntas

## Git

- Commits pequenos durante o módulo, com mensagem dizendo o que mudou
- Ao terminar um módulo: marcar o item acima, commit e `git push` para o GitHub
- Nunca commitar `.env` nem o conteúdo de `dados/`

## Cronograma — 2 semanas

Trabalho em tempo integral. Estudo à noite (1h30 a 2h) e nos fins de semana
(dia cheio).

**Semana 1 — base**

- Seg: ambiente, repositório no GitHub, .gitignore, estrutura de pastas
- Ter: abrir a URL da API no navegador e entender o retorno ANTES de codar.
  Testar intervalo de datas, série inexistente, período sem dado. Rascunho da
  ingestão
- Qua: ingestão funcionando — parâmetro de data, timeout, tratamento de erro,
  retry
- Qui: salvar em Parquet particionado (bronze)
- Sex: camada silver — tipagem (data em date, valor em decimal), deduplicação
- Sáb (dia cheio): Airflow no Docker, primeira DAG, tasks em sequência
- Dom (dia cheio): retry com backoff, idempotência, backfill de 2 anos

**Semana 2 — robustez e entrega**

- Seg: camada gold — séries lado a lado, variação diária e acumulada
- Ter: regras de qualidade, incluindo a de dia sem pregão
- Qua: a DAG para e avisa quando a qualidade falha
- Qui: um teste com pytest
- Sex: dashboard lendo da camada gold
- Sáb: README com diagrama da arquitetura, limpeza do repositório
- Dom: revisão geral e simulado de perguntas

## Checagem de fim de semana

Toda sexta, me faça 5 perguntas técnicas sobre o que construí na semana, no
nível de entrevista para Engenheiro de Dados Júnior. Não me dê as respostas —
só depois que eu tentar.

## Perguntas que preciso saber responder no final

- Por que camadas bronze/silver/gold?
- Por que Parquet e não CSV?
- Por que particionar por data?
- O que acontece se a API cair no meio da ingestão?
- Como garanto que rodar a DAG duas vezes não duplica dado?
- O que é backfill, como fiz o meu, e quando eu usaria na vida real?
- Como funciona retry com backoff e por que importa?
- Como eu detecto que o dado chegou errado antes de publicar?
- Como distingo ausência esperada de dado (feriado) de falha de ingestão?
- Como eu lidaria com uma série mensal e uma diária na mesma tabela?
