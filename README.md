# FICHA DE DECISÕES · War Room FiapBank (CP6 · 3 aulas)

> Este arquivo é o **README.md do repositório do grupo** (`cp6-warroom-<nome-do-grupo>`).
> Vale **5,0 pontos** (rodadas 0,5 · relâmpagos 0,3), e a nota é pela
> **justificativa**, não pela letra. Preencham após cada aula e commitem até
> **23h59 do mesmo dia** (regras completas na seção 5 do enunciado).
>
> **Os incidentes da madrugada são revelados só em aula.** O título de cada registro
> será **ditado pelo professor na hora**; ninguém se antecipe.

**Grupo (nome da equipe plantonista):** VegTrack

**Turma:** 2CCPW **Repo:** `cp6-warroom-vegtrack`

**Integrantes (nome + RM):**

| Nome | RM |
|---|---|
| Rogério Deligi Ferreira Filho | RM561942 |
| Maria Fernanda Garavelli Dantas | RM562686 |

**Convenção de commits** (1 commit por rodada; relâmpagos podem ir junto com a
rodada seguinte):

```
decisao: R1 - opcao C (<resumo da justificativa em uma frase>)
decisao: R2 - opcao A (<resumo em uma frase>)
pos-mortem: relatorio de incidente da madrugada
```

---

## 📁 Dossiê técnico do FiapBank (MVP em produção)

**Stack:** Java 17 + Spring Boot + Spring Data JPA + Oracle. API com endpoints em
`/api/contas` e `/api/transferencias` (cenário visto desde a Aula 13).

**Contrato e regras de negócio que o banco prometeu aos clientes e aos reguladores:**

| Regra | Como deve ser |
|---|---|
| Transferência **PIX** | taxa **R$ 0,00** |
| Transferência **TED** | taxa fixa **R$ 5,00** |
| Saldo | **nunca fica negativo**: transferência/saque sem saldo é recusado com erro claro |
| Número de conta | **sequencial e único** (1001, 1002, 1003...), gerado pelo sistema |
| Extrato de transferência | grava **quem pagou** e **quem recebeu**, na ordem certa |
| CPF | **dado sensível**: nunca aparece nas respostas da API |
| Consultas ao banco | sempre parametrizadas, e cada operação **usa e libera** a conexão |
| Suíte de testes | roda antes de todo deploy; **verde** é pré-requisito pra subir |

**Como escrever a justificativa:** nomeie o **mecanismo técnico** em jogo (o
conceito das Aulas 11 a 15 que explica o incidente) e o **trade-off** (velocidade ×
segurança × faturamento × dívida). "Porque é mais seguro" não é justificativa.

---

# 📝 REGISTRO DE DECISÕES

> A cada incidente, o professor dita o título (ex.: "Rodada 1"). **Copiem o modelo
> abaixo, colem no fim desta seção** e preencham com o rascunho feito em aula, junto
> com o **placar do grupo** após a consequência. Commitem 1 commit por rodada até
> 23h59 do dia.
>
> **Eventos relâmpago:** registrem apenas se o grupo for **afetado** (o professor
> chama quem for; não ser chamado é bom sinal).

**Modelo (copiar para cada decisão):**

```
## <título ditado pelo professor>

**Tipo:** ( rodada / relâmpago ) · **Voto:** ( A / B / C / D )

**Justificativa:**

<2 a 3 linhas: mecanismo técnico + trade-off>

**Placar do grupo após esta decisão:** 🔥 __ · 💰 R$ __ mil · 🧹 __
```

*(placar inicial do grupo: 🔥 7 · 💰 0 · 🧹 2)*

## Rodada 1: O saldo que virou negativo

**Tipo:** rodada · **Voto:** C

**Justificativa:**

O catch engole a SaldoInsuficienteException que protege a regra do dossiê: saldo nunca negativo. Remover esse tratamento permite recusar a operação com erro claro antes do débito. Escreveremos assertThrows e assertEquals para verificar a recusa e os saldos preservados. Isso consome tempo da janela, mas evita trocar proteção financeira por velocidade sem evidência contra regressão.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1

## Rodada 2: A injeção que derreteu o banco

**Tipo:** rodada · **Voto:** D

**Justificativa:**

Usaremos a derived query findByTitular do repository Spring Data JPA, substituindo o Statement com SQL concatenado por consulta com parâmetros vinculados, como exige o dossiê. O ciclo de recursos fica sob gestão do JPA, sem conexão JDBC manual abandonada. Integraremos e testaremos esse caminho antes de subir: custa validação agora, mas reduz código duplicado e exposição das contas na Black Friday.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 0

## Rodada 3: A noite das contas gêmeas

**Tipo:** rodada · **Voto:** D

**Justificativa:**

O getInstancia cria outro objeto e reinicia o contador, quebrando o Singleton. Guardar uma instância só protege uma JVM; escolheremos sequence Oracle para gerar números sequenciais entre instâncias, conforme o dossiê. A migração exige auditar duplicatas, iniciar a sequence acima do maior número existente e garantir unicidade no banco. Aceitamos esse trabalho antes da campanha para não depender de estado em memória para um identificador regulado.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 45 mil · 🧹 0

## Rodada 4: O comprovante que aponta pra pessoa errada

**Tipo:** rodada · **Voto:** B

**Justificativa:**

Os argumentos posicionais do construtor inverteram origem e destino. Corrigiremos a gravação e usaremos assertEquals para conferir pagador e recebedor, respeitando a ordem exigida pelo dossiê. Reprocessaremos somente registros cuja direção possa ser comprovada por evidência confiável, com rotina idempotente e sem refazer débitos. O custo de conciliação atrasa a entrega, mas deixar histórico falso transfere dívida e contestação ao cliente.

**Placar do grupo após esta decisão:** 🔥 9 · 💰 R$ 70 mil · 🧹 -1

---


## 🔎 O caminho do MEU grupo (preencher na 3ª aula, quando o mapa for revelado)

Uma linha por decisão registrada acima (usem os títulos ditados em aula):

| # | Decisão | Nossa letra | Consequência que ELA teria tido |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

*(adicionem linhas conforme as decisões da madrugada)*

---

# 📋 PÓS-MORTEM · relatório de incidente (montar em sala na 3ª aula)

> Rascunho em aula e commit final `pos-mortem:` até 23h59 do dia da 3ª aula.

## 1. Linha do tempo da madrugada

_______________________________________________________________________________________

_______________________________________________________________________________________

_______________________________________________________________________________________

## 2. Causa raiz de 2 incidentes (aula + mecanismo técnico)

**Incidente 1:** _______________________________________________

Aula/mecanismo:

_______________________________________________________________________________________

**Incidente 2:** _______________________________________________

Aula/mecanismo:

_______________________________________________________________________________________

## 3. O que faríamos diferente (2 rodadas + por quê)

_______________________________________________________________________________________

_______________________________________________________________________________________

## 4. A maior lição da equipe

_______________________________________________________________________________________
