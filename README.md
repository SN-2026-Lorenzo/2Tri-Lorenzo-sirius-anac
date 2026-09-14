# 1-A-A-2Tri-SEUNOME

Pipeline de dados: GitHub Actions → Supabase → GitHub Pages, com dados de
voos (SIROS/ANAC) para 41 aeroportos brasileiros.

> **Antes de tudo:** renomeie esta pasta e o repositório no GitHub para
> `1-A-A-2Tri-<SEUNOME>` (ex.: `1-A-A-2Tri-Lorenzo`), substituindo
> `SEUNOME` pelo seu nome, conforme o padrão da atividade.

## Estrutura

```
sql/setup.sql                          → schema completo (tabelas, RLS, GRANTs, 41 aeroportos)
scripts/fetch_flights.py               → busca voos do dia no SIROS/ANAC e faz upsert no Supabase
scripts/fetch_historico_anac.py        → importa o VRA mensal (histórico) da ANAC para o Supabase
.github/workflows/update-flights.yml   → workflow agendado (4x/dia) + manual — voos do dia
.github/workflows/importar-historico.yml → workflow mensal (dia 3) + manual — histórico VRA
data/airports.json                     → lista dos 41 aeroportos (usada pelo painel)
index.html                             → painel publicado no GitHub Pages (3 abas: voos do dia,
                                          histórico ANAC, pipeline)
docs/prompts-ia.md                     → documentação do uso de IA na atividade
docs/testes-complementacao/            → documentação das mudanças da complementação + prints dos testes
```

## Checklist da atividade

### Supabase (Organização SN-2026)

- [ ] Criar novo projeto no Supabase, dentro da organização **SN-2026**.
- [ ] Adicionar o usuário `rafael.ferques@ifpr.edu.br` ao projeto, com
      permissão **Developer** (Project Settings → Team).
- [ ] Abrir o **SQL Editor** e executar `sql/setup.sql` inteiro (idempotente
      — pode reexecutar sem perder dados).
- [ ] Conferir no fim da execução:
  - a query de `pg_tables` retornando `rowsecurity = true` para
    `aeroportos`, `voos`, `execucoes`, `historico_vra`;
  - `SELECT COUNT(*) FROM aeroportos` retornando **41**.
- [ ] Integração GitHub ↔ Supabase: em **Project Settings → Integrations →
      GitHub**, conectar e apontar para o repositório que você vai criar
      abaixo.
- [ ] Copiar as duas chaves do projeto (Project Settings → API):
  - **anon / public** (publishable key) → vai no `index.html`;
  - **service_role** (secret key) → vai **apenas** no GitHub Secret, nunca
    no código.
- [ ] Testar a API direto no navegador (troque `XXXX`, `SUA_ANON_KEY` e a
      data):
  ```
  https://XXXX.supabase.co/rest/v1/voos?icao_destino=eq.SBCA&data_referencia=eq.AAAA-MM-DD&order=chegada_iso.asc&limit=10&apikey=SUA_ANON_KEY
  ```

### GitHub

- [ ] Criar o repositório `1-A-A-2Tri-<SEUNOME>` na sua organização GitHub.
- [ ] Adicionar o usuário `GilFerques` à organização (se ainda não feito).
- [ ] Subir o conteúdo desta pasta para o novo repositório (é o "projeto
      base da última aula" já organizado).
- [ ] Ativar o **GitHub Pages** (Settings → Pages → Deploy from branch,
      apontando para `main` / raiz).
- [ ] Editar `index.html` e preencher `SUPABASE_URL` e `SUPABASE_ANON_KEY`
      com os valores reais do seu projeto (chave **anon**, nunca a
      `service_role`).
- [ ] Configurar os **Secrets** (Settings → Secrets and variables →
      Actions → Secrets):
  | Nome | Valor |
  |---|---|
  | `SUPABASE_URL` | Project URL do Supabase |
  | `SUPABASE_SERVICE_KEY` | chave `service_role` |
- [ ] Configurar a **Variable** (Settings → Secrets and variables →
      Actions → Variables):
  | Nome | Valor |
  |---|---|
  | `AIRPORTS` | `SBCA,SBCT,SBLO,SBMG,SBFI,SBFL,SBJV,SBNF,SBPA,SBCX,SBGR,SBSP,SBKP,SBRP,SBGL,SBRJ,SBCF,SBUL,SBMK,SBVT,SBBR,SBGO,SBCY,SBCG,SBSV,SBFZ,SBRF,SBSL,SBTE,SBJP,SBMO,SBSE,SBSG,SBEG,SBBE,SBSN,SBMQ,SBBV,SBPV,SBRB,SBPJ` |
- [ ] Executar o workflow manualmente (Actions → Pipeline SIROS → Supabase
      → Run workflow) e confirmar o ícone verde.
- [ ] Executar também o **Importar Histórico ANAC/VRA** manualmente (Actions
      → Importar Histórico ANAC/VRA → Run workflow), opcionalmente
      informando um `ano_mes` de teste (ex.: `2026-04`), e confirmar o
      ícone verde.
- [ ] Acessar a URL do GitHub Pages e validar as 3 abas do painel: **Voos
      do dia** (chegadas/partidas do aeroporto selecionado), **Histórico
      ANAC** (consulta por ICAO + mês) e **Pipeline** (log das últimas
      execuções).

### Documentação e envio (Class)

Envie via Class:

- [ ] Link do repositório GitHub.
- [ ] Link do painel publicado (GitHub Pages).
- [ ] Print do workflow com status verde (aba Actions).
- [ ] Confirmação de que a `service_role` / secret key está **apenas** no
      GitHub Secret (print das Secrets do repo, sem revelar o valor) e
      **não** aparece no `index.html` (print do arquivo publicado
      mostrando só a chave anon).
- [ ] Print do Table Editor do Supabase com voos inseridos na tabela `voos`.
- [ ] Print do resultado de:
  ```sql
  SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname='public';
  ```
- [ ] Print do resultado de:
  ```sql
  SELECT COUNT(*) FROM aeroportos; -- deve retornar 41
  ```
- [ ] URL da API REST retornando JSON no navegador.
- [ ] `docs/prompts-ia.md` preenchido com os prompts usados, versionado no
      repositório.

### Complementação (atividade adicional — Rafael Gil Ferques, 15/jun)

- [ ] Testar o workflow **Importar Histórico ANAC/VRA** e confirmar ícone
      verde no Actions.
- [ ] Testar a aba **Histórico ANAC** do painel com um ICAO e mês reais.
- [ ] Testar a aba **Pipeline** e confirmar que o log mostra as execuções
      recentes.
- [ ] Preencher `docs/testes-complementacao/README.md` com os resultados
      dos testes e anexar os prints nessa mesma pasta.

## Consulta auxiliar — voos por aeroporto hoje

```sql
SELECT
    icao_destino AS aeroporto,
    COUNT(*) AS chegadas_previstas,
    MIN(hr_chegada_utc::TEXT) AS primeiro_voo,
    MAX(hr_chegada_utc::TEXT) AS ultimo_voo
FROM voos
WHERE data_referencia = CURRENT_DATE
GROUP BY icao_destino
ORDER BY chegadas_previstas DESC;
```

## Glossário

- **ICAO** — código de 4 letras que identifica um aeroporto
  internacionalmente. Ex.: `SBCA` = Cascavel.
- **IATA** — código de 3 letras do aeroporto, usado em passagens aéreas.
  Ex.: `CAC` = Cascavel.
- **anon key** — chave pública do Supabase; permite leitura pelo navegador,
  pode aparecer no código (`index.html`).
- **service_role key** — chave secreta do Supabase; acesso total, nunca
  pode aparecer em código público — fica só no GitHub Secret.
