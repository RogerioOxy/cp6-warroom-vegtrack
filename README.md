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

## Rodada 5: A fábrica que devolve null

**Tipo:** rodada · **Voto:** A

**Justificativa:**

A Factory deve criar um objeto válido ou lançar exceção clara para tipo não suportado, em vez de devolver null e falhar longe da causa no save do repository. Testaremos esse contrato com assertThrows. O fail-fast não implementa a conta UNIVERSITARIA anunciada: aceitamos a recusa explícita e o impacto no faturamento até definir esse produto, preservando a causa do erro sem cadastrar outra modalidade silenciosamente.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 -1

## Evento relâmpago C: O cadastro com metade dos campos

**Tipo:** relâmpago · **Voto:** B

**Justificativa:**

O Builder dará valores padrão só a campos opcionais sem efeito de negócio, como saldo inicial zero. Tipo de conta e CPF continuam obrigatórios; o build() lançará exceção clara quando faltarem, coerente com o fail-fast da R5. A landing precisa enviar o tipo de conta e exigir o CPF, que nunca vai para a resposta da API, conforme o dossiê. Tipo não suportado continua sujeito à recusa da Factory. Aceitamos perder conversão na landing ao exigir esses dados, em troca de não criar contas incompletas nem espalhar null.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 -1

## Rodada 6: O teste vermelho às 03h45

**Tipo:** rodada · **Voto:** B

**Justificativa:**

Investigaremos o número mágico 50.00 e corrigiremos a taxa TED para R$ 5,00, preservando o assertEquals do contrato e verificando também PIX a R$ 0,00. @Disabled ou mudar o esperado esconderia a divergência. Rodaremos a suíte completa antes do deploy, como exige o dossiê, mesmo perdendo a janela. Aceitamos atrasar as correções e a receita; a violação de saldo da R1 continua urgente, sem justificar liberar outra cobrança incorreta.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 100 mil · 🧹 -2

## Rodada 7: Go/No-Go da Black Friday

**Tipo:** rodada · **Voto:** B

**Justificativa:**

A concentração de responsabilidades no TransferenciaService exige separar geração de número, cálculo de taxa e montagem do extrato, mantendo no serviço a coordenação da transferência e a recusa por saldo insuficiente. Escolhemos abrir duas horas depois, condicionado às correções e à suíte verde real: assertEquals e assertThrows para taxas, saldo e extrato, além de integração para a sequence. Atrasar custa faturamento, mas cumprir o dossiê reduz o risco de repetir falhas sob maior volume.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 200 mil · 🧹 -3

---

## 🔎 O caminho do MEU grupo

| # | Decisão | Nossa letra | Consequência que ELA teve |
|---|---|---|---|
| 1 | R1 · O saldo que virou negativo | C | Levamos 40 min a mais, com 8 clientes afetados no total, mas incluímos o assertThrows que protege a invariante: reputação +1, prejuízo +R$ 35 mil, dívida -1; 🔥 8 · 💰 R$ 35 mil · 🧹 1. |
| 2 | R2 · A injeção que derreteu o banco | D | Usamos a query pronta do repository, com cerca de 50 min de trabalho, mais que o PreparedStatement: reputação +0, prejuízo +R$ 0 mil, dívida -1; 🔥 8 · 💰 R$ 35 mil · 🧹 0. |
| 3 | R3 · A noite das contas gêmeas | D | Assumimos cerca de 1h de migração para o banco garantir a unicidade e gerar a numeração: reputação +0, prejuízo +R$ 10 mil, dívida +0; 🔥 8 · 💰 R$ 45 mil · 🧹 0. |
| 4 | R4 · O comprovante que aponta pra pessoa errada | B | Corrigimos a gravação e reprocessamos o histórico em cerca de 2h, deixando os comprovantes já emitidos certos: reputação +1, prejuízo +R$ 25 mil, dívida -1; 🔥 9 · 💰 R$ 70 mil · 🧹 -1. |
| 5 | R5 · A fábrica que devolve null | A | A Factory passou a rejeitar na origem com "Tipo de conta inválido: UNIVERSITARIA", em cerca de 30 min: reputação -1, prejuízo +R$ 0 mil, dívida +0; 🔥 8 · 💰 R$ 70 mil · 🧹 -1. |
| 6 | ⚡C · O cadastro com metade dos campos | B | O Builder deu valores padrão só a campos opcionais sem efeito de negócio, como saldo inicial zero; tipo de conta e CPF obrigatórios, com exceção clara no build() quando faltam, coerente com o fail-fast da R5. A landing precisa enviar o tipo e exigir o CPF, que nunca vai para a resposta da API, conforme o dossiê. A landing perde conversão ao exigir esses dados, em troca de não criar contas incompletas nem espalhar null: reputação +0, prejuízo +R$ 0 mil, dívida +0; 🔥 8 · 💰 R$ 70 mil · 🧹 -1. |
| 7 | R6 · O teste vermelho às 03h45 | B | Perdemos a janela e as correções só entraram depois da abertura, mas a investigação confirmou o erro no código e a taxa contratual de R$ 5: reputação -1, prejuízo +R$ 30 mil, dívida -1; 🔥 7 · 💰 R$ 100 mil · 🧹 -2. |
| 8 | R7 · Go/No-Go da Black Friday | B | A suíte saudável sustentou a subida às 10h, com perda do início do pico: reputação +1, prejuízo +R$ 100 mil, dívida -1; 🔥 8 · 💰 R$ 200 mil · 🧹 -3. |

**Perfil final da noite (congelado):** A Noite dos Heróis · 🔥 8 · 💰 R$ 200 mil · 🧹 -3

Para o banco, esse perfil significa que abriu com a confiança dos clientes preservada e o time pôde dormir quase em paz, apesar do prejuízo e do atraso.

---

# 📋 PÓS-MORTEM · relatório de incidente

Grupo VegTrack: Rogério Deligi Ferreira Filho, RM561942, e Maria Fernanda Garavelli Dantas, RM562686. Turma 2CCPW, Programação Orientada a Objetos em Java, FIAP. CP6: War Room FiapBank.

## 1. Linha do tempo da madrugada

Partimos de 🔥 7 · 💰 0 · 🧹 2. O caminho vivido foi C, D, D, B, A, B, B, B, com o relâmpago ⚡C entre R5 e R6.

| Relógio | Incidente | O que o grupo decidiu | Efeito |
|---|---|---|---|
| 19h12 | R1 · O saldo que virou negativo | C: retirar o catch que engolia SaldoInsuficienteException e testar a recusa antes do débito, preservando os saldos. | Mais 40 min, 8 clientes afetados no total e um assertThrows honesto; reputação +1, prejuízo +R$ 35 mil, dívida -1; 🔥 8 · 💰 R$ 35 mil · 🧹 1. |
| 20h10 | ⚡A · Comparação de Strings com ==, Aula 2 | Não vivido: exigia rollback na R1; escolhemos C. | A escolha anterior evitou esse ramo, que confrontaria a comparação por referência com a derived query pronta da Aula 13; nenhum efeito adicional no placar. |
| 21h02 | R2 · A injeção que derreteu o banco | D: substituir SQL concatenado por findByTitular do repository Spring Data JPA, com parâmetros vinculados, Aulas 12 e 13. | Cerca de 50 min, mais que o PreparedStatement; reputação +0, prejuízo +R$ 0 mil, dívida -1; 🔥 8 · 💰 R$ 35 mil · 🧹 0. |
| 22h40 | ⚡B · Conexão sem close(), Aula 12 | Não vivido: exigia PreparedStatement na R2; escolhemos D, com recursos sob gestão do JPA. | A escolha anterior evitou esse ramo de conexão JDBC manual, cuja correção envolveria try-with-resources; nenhum efeito adicional no placar. |
| 22h50 | R3 · A noite das contas gêmeas | D: usar sequence Oracle e unicidade no banco, auditando duplicatas e iniciando a sequence acima do maior número existente, Aula 14. | Cerca de 1h de migração; reputação +0, prejuízo +R$ 10 mil, dívida +0; 🔥 8 · 💰 R$ 45 mil · 🧹 0. |
| 00h47 | R4 · O comprovante que aponta pra pessoa errada | B: corrigir origem e destino na entidade gravada e reprocessar o histórico, Aula 13, com evidência confiável, idempotência e sem refazer débitos. | Cerca de 2h, gravação corrigida e comprovantes antigos certos; reputação +1, prejuízo +R$ 25 mil, dívida -1; 🔥 9 · 💰 R$ 70 mil · 🧹 -1. |
| 01h30 | R5 · A fábrica que devolve null | A: lançar exceção clara para tipo não suportado, em vez de deixar null chegar ao save, Aulas 8, 11 e 14. | Cerca de 30 min; erro "Tipo de conta inválido: UNIVERSITARIA" na origem, sem implementar o produto anunciado; reputação -1, prejuízo +R$ 0 mil, dívida +0; 🔥 8 · 💰 R$ 70 mil · 🧹 -1. |
| 02h20 | ⚡C · O cadastro com metade dos campos | B: Builder com valores padrão só para campos opcionais sem efeito de negócio, como saldo inicial zero, Aula 14; tipo de conta e CPF obrigatórios, com exceção clara no build() quando faltam, coerente com o fail-fast da R5. A landing precisa enviar o tipo e exigir o CPF, que nunca vai para a resposta da API, conforme o dossiê. | A landing perde conversão ao exigir esses dados, em troca de não criar contas incompletas nem espalhar null, mantendo a recusa da Factory para tipo não suportado; reputação +0, prejuízo +R$ 0 mil, dívida +0; 🔥 8 · 💰 R$ 70 mil · 🧹 -1. |
| 03h20 | ⚡D · Teste sem assert, Aula 15 | Não vivido: exigia A ou C na R2; escolhemos D. | A escolha anterior evitou esse ramo de teste sem verificação, sem provar por si só a qualidade de qualquer suíte; nenhum efeito adicional no placar. |
| 03h45 | R6 · O teste vermelho às 03h45 | B: investigar 50.00, corrigir a TED para R$ 5,00, conferir PIX a R$ 0,00 e preservar o assertEquals do contrato, Aula 15. | Janela perdida, correções depois da abertura e suíte saudável; reputação -1, prejuízo +R$ 30 mil, dívida -1; 🔥 7 · 💰 R$ 100 mil · 🧹 -2. |
| 05h58 | ⚡E · Taxa 5 ou 50 | Não vivido: exigia suíte doente na R6; escolhemos B e corrigimos o código. | A escolha anterior evitou esse ramo porque o contrato de R$ 5 permaneceu protegido pelo teste; nenhum efeito adicional no placar. |
| 06h15 | R7 · Go/No-Go da Black Friday | B: abrir duas horas depois, condicionado às correções e à suíte verde, separando numeração, taxas e extrato da coordenação no TransferenciaService de 80 linhas. | Subida sólida às 10h, perdendo o início do pico; reputação +1, prejuízo +R$ 100 mil, dívida -1; 🔥 8 · 💰 R$ 200 mil · 🧹 -3. |

O perfil final foi A Noite dos Heróis, com suíte saudável ao amanhecer.

## 2. Causa raiz de 2 incidentes (aula + mecanismo técnico)

Os trechos a seguir são a correção proposta e o teste de regressão de cada causa raiz.

### R1 · Exceção engolida pelo catch, Aula 11

**Sintoma observado:** saldo negativo, contrariando a invariante financeira de que o saldo nunca pode ficar negativo.

**Causa raiz:** o próprio método capturava a SaldoInsuficienteException que acabara de lançar e engolia a recusa. O tratamento local neutralizava a proteção e permitia continuar o fluxo que deveria ser interrompido antes do débito.

**Por que passou despercebido:** a captura escondia a falha de quem chamava o método. Um teste que apenas executasse a transferência não distinguiria operação concluída de recusa ignorada. A inclusão do assertThrows na R1 expõe essa diferença e precisa ser acompanhada da conferência dos saldos, pois lançar uma exceção depois de alterar valores também violaria a regra.

**Correção que fecha essa classe do bug:** validar o saldo antes de qualquer alteração e propagar a exceção de domínio, sem um catch que transforme a recusa em continuidade. A proteção abaixo cobre saldo insuficiente; concorrência e falhas de persistência exigem garantias transacionais próprias.

```java
public void transferir(Conta origem, Conta destino, BigDecimal valor) {
    if (origem.getSaldo().compareTo(valor) < 0) {
        throw new SaldoInsuficienteException();
    }
    origem.debitar(valor);
    destino.creditar(valor);
}
```

**Teste que impede a regressão:** exigir a exceção e verificar que nenhuma conta foi alterada na recusa.

```java
@Test
void recusaSaldoInsuficienteSemAlterarContas() {
    var origem = new Conta(new BigDecimal("5.00"));
    var destino = new Conta(BigDecimal.ZERO);
    var servico = new TransferenciaService();
    assertThrows(SaldoInsuficienteException.class,
        () -> servico.transferir(origem, destino, new BigDecimal("50.00")));
    assertEquals(new BigDecimal("5.00"), origem.getSaldo());
    assertEquals(BigDecimal.ZERO, destino.getSaldo());
}
```

### R6 · Número mágico contra o contrato do teste, Aula 15

**Sintoma observado:** o teste vermelho apontava divergência entre a taxa TED do código, 50.00, e o contrato, R$ 5,00.

**Causa raiz:** o cálculo embutia um número mágico incompatível com a regra contratual. Usar @Disabled ou trocar o esperado para acompanhar o erro retiraria o alarme sem corrigir a cobrança.

**Por que passou despercebido:** um cálculo pode executar sem exceção e ainda cobrar errado. A divergência só fica observável quando o resultado é confrontado com uma expectativa independente, o contrato. A investigação da R6 mostrou que o código precisava mudar; o registro não permite atribuir a falha a uma execução anterior específica da suíte.

**Correção que fecha essa classe do bug:** concentrar a regra de taxa em uma calculadora, com valores monetários explícitos em BigDecimal e cobertura dos contratos TED e PIX. Separar a responsabilidade facilita conferir a regra sem percorrer toda a transferência.

```java
class CalculadoraTaxa {
    BigDecimal calcular(TipoTransferencia tipo) {
        return switch (tipo) {
            case TED -> new BigDecimal("5.00");
            case PIX -> new BigDecimal("0.00");
        };
    }
}
```

**Teste que impede a regressão:** manter os esperados contratuais, sem @Disabled. O exemplo considera os tipos TED e PIX; a integração deve conferir também a taxa efetivamente usada pelo serviço e registrada no extrato.

```java
@Test
void respeitaTaxasContratuaisDeTedEPix() {
    var calculadora = new CalculadoraTaxa();
    assertEquals(new BigDecimal("5.00"),
        calculadora.calcular(TipoTransferencia.TED));
    assertEquals(new BigDecimal("0.00"),
        calculadora.calcular(TipoTransferencia.PIX));
}
```

## 3. O que faríamos diferente (2 rodadas + por quê)

### R3 · Preparar a migração antes da janela

A escolha D deu ao banco a responsabilidade pela numeração, mas custou cerca de 1h e R$ 10 mil em plena madrugada. Hoje anteciparíamos a auditoria de duplicatas, o ensaio da migração e a validação da unicidade entre instâncias para antes da campanha, deixando o avanço da sequence acima do maior número existente como condição de liberação. Continuaríamos com D: corrigir apenas o Singleton não resolve a numeração entre JVMs. O trade-off seria retirar tempo de outras entregas para preparar a migração; não há base para prometer eliminação do atraso ou do prejuízo. Nossa falha de condução foi chegar à janela sem essa preparação.

### R4 · Separar a correção de novas gravações do histórico

A escolha B restaurou a integridade dos comprovantes, mas o trabalho conjunto custou cerca de 2h e R$ 25 mil. Hoje corrigiríamos primeiro a gravação de origem e destino e validaríamos pagador e recebedor; depois conduziríamos o reprocessamento do histórico em etapa própria, com evidência confiável, idempotência e sem refazer débitos. Continuaríamos responsáveis por concluir o histórico, sem declarar integridade total antes da conciliação. O trade-off seria manter temporariamente comprovantes antigos incorretos e a possibilidade de contestação, enquanto novas gravações já estariam protegidas.

## 4. A maior lição da equipe

Aprendemos que preservar o contrato do banco exige tornar a falha visível e verificar o estado que fica depois dela. Na R1, uma exceção sem propagação não protegia o saldo; na R6, executar sem erro não garantia a taxa correta. Aceitamos atrasos para corrigir essas violações, mas o prejuízo de R$ 200 mil mostra o custo de concentrar validação e mudanças estruturais na madrugada da campanha. A responsabilidade inclui preparar essas verificações antes da janela. **Integridade**

### Apresentação relâmpago (roteiro de 2 a 3 min)

- **Caminho:** R1 C, R2 D, R3 D, R4 B, R5 A, ⚡C B, R6 B e R7 B; A Noite dos Heróis, 🔥 8 · 💰 R$ 200 mil · 🧹 -3, com suíte saudável.
- **Pior momento:** na R6, descobrir a cobrança TED de 50.00 contra o contrato de R$ 5,00 e perder a janela para corrigir o código sem silenciar o teste.
- **Maior lição:** uma regra só está protegida quando a falha aparece e o teste verifica o resultado e o estado preservado, incluindo os saldos na recusa.
- **O que mudaríamos:** antecipar a preparação da migração da R3 e separar, na R4, a correção de novas gravações do reprocessamento acompanhado do histórico, reconhecendo os custos e riscos.
- **A palavra:** **Integridade**.
