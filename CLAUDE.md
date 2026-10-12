# Monitor de Preços — Pipeline de Dados

## Sobre este projeto

Pipeline que coleta preços e disponibilidade de produtos da API pública do
Mercado Livre, processa em camadas (bronze/silver/gold) e orquestra com
Airflow.

**O objetivo é APRENDER engenharia de dados.** Sou Analista de Dados buscando
vaga de Engenheiro de Dados Júnior e preciso saber explicar cada parte deste
projeto numa entrevista técnica. Um repositório que eu não sei defender é pior
do que nenhum.

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

**bronze** — resposta crua da API em Parquet, particionada por data. Sem
transformação. Se a API mudar, o original continua aqui.

**silver** — dado limpo e tipado: sku, titulo, preco, vendedor, disponivel,
data_coleta. Deduplicado por chave.

**gold** — tabelas de análise: variação de preço por produto, ranking de
vendedor, histórico diário.

**Airflow** — DAG diária, três tasks em sequência (ingestão → silver → gold),
com retry e idempotência.

**Qualidade** — antes de promover silver → gold: volume de linhas dentro do
esperado, nenhum preço nulo ou negativo, nenhuma chave duplicada. Se falhar, a
DAG para e avisa.

## Estrutura

```
monitor-precos/
├── src/
│   ├── ingestao.py
│   ├── transformacao.py
│   ├── qualidade.py
│   └── config.py
├── dags/
│   └── monitor_precos_dag.py
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

## Cronograma — 2 semanas

Trabalho em tempo integral. Estudo à noite (1h30 a 2h) e nos fins de semana
(dia cheio).

**Semana 1 — base**

- Seg: ambiente, repositório no GitHub, .gitignore, estrutura de pastas
- Ter: entender a API no navegador antes de codar (paginação, formato do
  JSON), rascunho da ingestão
- Qua: ingestão funcionando com tratamento de erro, timeout e paginação
- Qui: salvar em Parquet particionado por data (bronze)
- Sex: camada silver — limpeza, tipagem, deduplicação
- Sáb (dia cheio): Airflow no Docker, primeira DAG, tasks em sequência
- Dom (dia cheio): retry, idempotência, backfill

**Semana 2 — robustez e entrega**

- Seg: camada gold — variação de preço, ranking
- Ter: regras de qualidade entre silver e gold
- Qua: a DAG para e avisa quando a qualidade falha
- Qui: um teste com pytest
- Sex: dashboard lendo da camada gold
- Sáb: README com diagrama da arquitetura, limpeza do repositório
- Dom: revisão geral e simulado de perguntas

## Módulos

Cada módulo segue o mesmo ciclo: explicação do conceito → esqueleto pronto
(imports, log, assinaturas) → eu escrevo só o miolo da lógica → revisão → eu
explico com minhas palavras → commit e push.

- [x] **0 — Ambiente**: Git, .gitignore, estrutura de pastas
- [ ] **1 — Ingestão**: chamada à API com timeout, tratamento de erro e paginação
- [ ] **2 — Bronze**: resposta crua em Parquet particionado por data
- [ ] **3 — Silver**: limpeza, tipagem, deduplicação
- [ ] **4 — Airflow**: Docker, primeira DAG, três tasks em sequência
- [ ] **5 — Robustez**: retry com backoff, idempotência, backfill
- [ ] **6 — Gold**: variação de preço, ranking de vendedor, histórico diário
- [ ] **7 — Qualidade**: regras entre silver e gold; a DAG para e avisa
- [ ] **8 — Teste**: um teste com pytest
- [ ] **9 — Dashboard**: leitura da camada gold
- [ ] **10 — Entrega**: README com diagrama, limpeza, simulado de perguntas

## Git

- Commits pequenos durante o módulo, com mensagem dizendo o que mudou
- Ao terminar um módulo: marcar o item acima, commit e `git push` para o GitHub
- Nunca commitar `.env` nem o conteúdo de `dados/`

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
- O que é backfill e quando eu usaria?
- Como funciona retry com backoff e por que importa?
- Como eu detecto que o dado chegou errado antes de publicar?
