# Checkpoint 5 — Bug Hunt PetFiap

Disciplina: Programação Orientada a Objetos · Projeto: PetFiap — pet shop & clínica veterinária

## Identificação

**Grupo:** `___` *(preencher com o nome do grupo usado no repositório)*

| Integrante | RM | Turma |
|---|---|---|
| Lucas M. | 563667 | `2CCPO` |
| Kaio Correa | 56433 | `2CCPO` |
| Gustavo Braga | 562247 | `2CCPO` |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final** | 26 testes, 0 falhas |

**Estado de entrega original:** 20 testes, 9 falhas.

**Divisão do trabalho:** a coluna *Resp.* indica quem assinou o commit de cada item.

---

## Parte 1 — Bugs encontrados

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina | Resp. |
|---|---|---|---|---|---|
| bug01 | `deveManterUmaUnicaInstancia` falha no `assertSame` com duas referências diferentes (`GeradorProtocolo@2f5ac102` × `@5df778c3`); `deveGerarProtocolosSequenciais` passa na 1ª asserção e quebra na 2ª com `expected: <2> but was: <1>` — cada chamada devolve um contador zerado. O console imprime `GeradorProtocolo criado!` uma vez por chamada | `GeradorProtocolo.getInstancia()` (model, ~l.18) faz `return new GeradorProtocolo()` sem nunca atribuir o campo `instancia`, que permanece `null` para sempre | Atribuir `instancia = new GeradorProtocolo()` antes de retornar, e retornar sempre o campo. Método marcado como `synchronized` para honrar o comentário de thread-safety | Padrão Singleton (Aula 14): instância única guardada em campo estático | Lucas |
| bug02 | `AtendimentoFactoryTest.deveCriarTosaQuandoTipoForTosa` falha: esperava `Tosa`, veio `Banho` | `AtendimentoFactory.criar()` (~l.17): o `case "TOSA"` instancia `new Banho(...)` — caso trocado no `switch` | Trocar para `new Tosa(...)` no ramo `"TOSA"` | Padrão Factory (Aula 14): o único ponto que conhece as subclasses concretas | Lucas |
| bug03 | `AtendimentoFactoryTest.devePreencherOsDadosDoPetNaConsulta` falha com `expected: <Mimi> but was: <null>` | `ConsultaVeterinaria` (~l.17): o construtor recebe os cinco parâmetros mas chama `super()` sem repassá-los, então nome, porte, tutor, data e status ficam nulos | Chamar `super(protocolo, petNome, petPorte, tutorNome, dataHora)` | Herança e encadeamento de construtores (Aula 7) | Lucas |
| bug04 | `AtendimentoBuilderTest.deveMontarAtendimentoCompleto` falha com `expected: <Rex> but was: <null>` | `AtendimentoBuilder.comPet()` (~l.24): `petNome = petNome;` atribui o parâmetro a si mesmo — falta o `this.`, e o campo nunca é preenchido | Trocar por `this.petNome = petNome;` | Escopo de variáveis e a referência `this` (Aula 6) | Lucas |
| bug05 | `deveRecusarMontagemSemNomeDoPet` e `deveRecusarMontagemSemPorte` falham: nenhuma exceção é lançada e o objeto inválido nasce | `AtendimentoBuilder.construir()` (~l.41) delega direto para a factory sem validar nada; o comentário acima afirma que a validação é do controller | Validar `petNome`, `petPorte`, `tutorNome` e `dataHora` no `construir()`, lançando `IllegalArgumentException`. Comentário enganoso corrigido | Padrão Builder (Aula 14): o objeto só nasce em estado válido | Lucas |
| bug06 | Nenhum teste entregue cobria a duração da tosa; o teste novo `teste03` nasceu vermelho esperando 60 e recebendo 30 | `Tosa` (~l.40): `public int getDuracaoMinutos(String porte)` cria uma **sobrecarga** nova em vez de sobrescrever o método sem argumentos da superclasse, que continua devolvendo 30 | Remover o parâmetro, deixando `public int getDuracaoMinutos()` com `@Override` | Sobrescrita × sobrecarga e o papel de `@Override` (Aula 7) | Lucas |
| bug07 | `deveRecusarAgendamentoComHorarioJaOcupado` falha com `Unexpected exception type thrown — expected HorarioOcupadoException, actual NullPointerException`. O conflito não é detectado, a execução segue para `repository.save()`, que no mock não estubado devolve `null`, e a linha 30 estoura em `salvo.getProtocolo()` | `AgendaService.agendar()` (~l.23): compara `a.getPetNome() == novo.getPetNome()` — identidade de referência em vez de igualdade de valor | Trocar por `.equals()` (ou `Objects.equals`) | `==` × `.equals()` (Aula 7) | `___` |
| bug08 | Mesmo teste e mesmo `NullPointerException` do bug07: mesmo com os nomes iguais por sorte, a data nunca casa e a condição inteira fica falsa | `AgendaService.agendar()` (~l.23): `a.getDataHora() == novo.getDataHora()` compara duas instâncias distintas de `LocalDateTime` com valores iguais | Trocar por `.equals()` | Igualdade de objetos em tipos do `java.time` (Aula 7) | `___` |
| bug09 | `AgendaServiceTest.deveLancarExcecaoQuandoAtendimentoNaoExiste` falha: esperava `AtendimentoNaoEncontradoException` e o método devolveu `null` | `AgendaService.buscarPorId()` (~l.40): um `catch (Exception e)` genérico engole a própria exceção lançada pelo `orElseThrow` e retorna `null` | Remover o try/catch e deixar a exceção subir até o controller | Tratamento de exceções (Aula 11): não engolir exceção nem devolver `null` silencioso | `___` |
| bug10 | Regra sem cobertura; o teste novo `teste04` nasceu vermelho: banho de porte pequeno cobrou R$ 100,00 em vez de R$ 60,00 | `Banho.calcularPreco()` (~l.27): a tabela de preços está invertida — `PEQUENO` devolve 100,00 e `GRANDE` devolve 60,00 | Inverter para `PEQUENO` = 60,00, `MEDIO` = 80,00, `GRANDE` = 100,00 | Polimorfismo: a regra de preço mora na subclasse (Aula 7) | `___` |
| bug11 | Regra sem cobertura; o teste novo `teste05` nasceu vermelho: cancelar um atendimento já concluído foi aceito | `Atendimento.cancelar()` (~l.63) atribui `CANCELADO` sem checar o status atual, enquanto `concluir()` valida corretamente | Validar que o status é `AGENDADO` e lançar `StatusInvalidoException` caso contrário | Encapsulamento: a regra de transição de estado mora no model | `___` |
| bug12 | Regra sem cobertura; o teste novo `teste06` nasceu vermelho: agendamento com data no passado foi aceito e chegou a chamar o repositório | Validação inexistente — nem `AtendimentoBuilder.construir()` nem `AgendaService.agendar()` comparam `dataHora` com o instante atual | Recusar no `construir()` com `IllegalArgumentException` antes de qualquer acesso ao banco | Validação de entrada e falha rápida (Aula 11) | `___` |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei | Resp. |
|---|---|---|---|---|
| clean01 | `AtendimentoFactory.criar(int p, String t, String n, String po, String tu, LocalDateTime d)` | Nomes sem significado: a assinatura obriga quem lê a decodificar cada letra e facilita trocar a ordem dos argumentos | Renomeados para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome`, `dataHora` | Lucas |
| clean02 | `System.out.println` no construtor do `GeradorProtocolo` e no `AgendaService.agendar()` | Saída de diagnóstico impressa direto no console: polui a execução dos testes, não tem nível de severidade e não é configurável | Removida a do Singleton e substituída a do service por um `org.slf4j.Logger` em nível `INFO`, com parâmetros `{}` — o SLF4J já vem no `spring-boot-starter`, sem alterar o `pom.xml` | Lucas |
| clean03 | `AtendimentoController.calcularDescontoFidelidade()` e o bloco de comentários "Fidelidade (futuro)" | Código morto: método privado nunca chamado, mantido "para o futuro" — o histórico do Git já guarda o que foi apagado | Método e comentário removidos | Lucas |
| clean04 | `@Autowired` em campo no `AgendaService` e no `AtendimentoController` | Injeção por campo esconde as dependências, impede `final` e dificulta instanciar a classe em teste sem framework | Trocada por injeção via construtor, com os campos `final` | `___` |
| clean05 | Literais `"AGENDADO"`, `"CONCLUIDO"` e `"CANCELADO"` espalhados por `Atendimento`, `AgendaService` e testes | Números/strings mágicos repetidos: um erro de digitação não é pego pelo compilador | Extraídos para constantes `public static final` em `Atendimento` (ou `enum StatusAtendimento`) | `___` |
| clean06 | Comentários de `AgendaService.buscarPorId` ("nunca retorna null, o orElseThrow garante a excecao"), de `AtendimentoBuilder.construir` e de `GeradorProtocolo` ("Thread-safe") | Comentários que mentem sobre o código: os três descreviam um comportamento que o código não tinha | Comentários corrigidos para descrever o que o código passou a fazer de fato | `___` |

## Parte 3 — Testes novos (regras que estavam sem cobertura)

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) | Resp. |
|---|---|---|---|---|
| teste01 | `ConsultaVeterinariaTest.deveCustar150ReaisIndependenteDoPorte` | Consulta custa R$ 150,00 fixo — o porte não muda o preço | **Verde de cara** — a regra já estava correta; o teste passa a proteger contra regressão | Lucas |
| teste02 | `AgendaServiceTest.deveRecusarConclusaoDeAtendimentoCancelado` | `concluir()` em atendimento `CANCELADO` deve recusar com `StatusInvalidoException` | **Verde de cara** — `concluir()` já validava `!"AGENDADO".equals(status)` | Lucas |
| teste03 | `TosaTest.deveDurar60Minutos` | Tosa dura 60 minutos | **Vermelho** — revelou o bug06 (sobrecarga em vez de sobrescrita); veio 30 | Lucas |
| teste04 | `BanhoTest.deveCobrarPrecoPorPorte` | Banho: R$ 60,00 (pequeno), R$ 80,00 (médio), R$ 100,00 (grande) | **Vermelho** — revelou o bug10 (tabela de preços invertida) | `___` |
| teste05 | `AgendaServiceTest.deveRecusarCancelamentoDeAtendimentoConcluido` | `cancelar()` em atendimento `CONCLUIDO` deve recusar — atendimento já realizado | **Vermelho** — revelou o bug11 (`cancelar()` sem validação de status) | `___` |
| teste06 | `AtendimentoBuilderTest.deveRecusarAgendamentoComDataNoPassado` | Agendar com data/hora no passado deve recusar com `IllegalArgumentException`, sem consultar o banco | **Vermelho** — revelou o bug12 (validação inexistente) | `___` |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)

As mensagens de falha funcionaram como endereço do bug, não só como aviso. O `expected: <Rex> but was: <null>` do `AtendimentoBuilderTest` dizia que o objeto tinha nascido sem nome — não que o getter estivesse errado. Isso reduziu a busca ao caminho que preenche o nome, e lá estava o `petNome = petNome` sem `this`. O mesmo sintoma apareceu no `devePreencherOsDadosDoPetNaConsulta`, mas com outra causa: o `super()` sem argumentos. Dois testes com mensagem quase idêntica e bugs diferentes — foi preciso ler o código, não só a mensagem.

A vantagem sobre testar com `curl` é o isolamento e o custo. A suíte roda em segundos, sem banco e sem subir o Spring, e aponta a classe exata; com `curl` só veríamos um JSON com campo nulo, sem saber se a culpa é do controller, do builder, da factory ou do model. Além disso a suíte é repetível: depois de cada correção rodamos tudo de novo e vimos se alguma correção quebrou outra regra.

### 2. Mock e injeção de dependência (Aulas 13 a 15)

Em produção quem injeta é o container do Spring: o `AtendimentoRepository` é uma interface que estende `JpaRepository`, o Spring gera a implementação em tempo de execução e a coloca no `AgendaService` por causa do `@Autowired`. No teste não existe container, então quem injeta é o Mockito: o `@Mock` cria um dublê do `AtendimentoRepository` e o `@InjectMocks` o coloca dentro do `AgendaService`, com a `MockitoExtension` fazendo o papel do ciclo de vida.

O teste roda sem banco porque o service nunca dependeu de Oracle — depende da *interface* `AtendimentoRepository`. Essa inversão de dependência é o que permite trocar a implementação real por uma falsa sem alterar uma linha do service. O `when(repository.findById(1L)).thenReturn(...)` define a resposta que interessa ao cenário, e o `verify(repository, never()).save(any())` prova que o service recusou a operação antes de gravar — algo que um teste com banco real teria dificuldade de afirmar com a mesma precisão.

### 3. `==` vs `.equals()` (Aula 7)

`==` compara referências: pergunta se as duas variáveis apontam para o mesmo objeto na memória. `.equals()` compara conteúdo, do jeito que a classe definiu. No `AgendaService.agendar()` a verificação de conflito usava `==` nas duas pontas, e por isso o agendamento duplicado passava batido.

O detalhe cruel é que a comparação de nomes *parecia* funcionar. Como `"Rex"` aparece como literal nos dois objetos do teste, o Java reaproveita a mesma instância do pool de strings e `==` devolve `true` por sorte. Já a data não tem esse privilégio: o teste cria a segunda data com `LocalDateTime.parse(...)`, gerando um objeto novo com o mesmo valor, e `==` devolve `false`. Resultado: a condição inteira ficava falsa e o conflito nunca era detectado. Trocando as duas comparações por `.equals()`, passamos a comparar valor em ambos os casos, e o comportamento deixa de depender de onde a string foi criada.

### 4. Sobrescrita vs sobrecarga (Aula 7)

`Atendimento` declara `public int getDuracaoMinutos()` devolvendo 30. A `Tosa` escreveu `public int getDuracaoMinutos(String porte)` devolvendo 60. Como a lista de parâmetros é diferente, isso não é override — é **overload**: um método novo, que coexiste com o herdado. Ninguém chama a versão com parâmetro, então toda tosa continuava informando 30 minutos, e o código compilava sem um aviso sequer.

Com `@Override` o compilador verifica se existe mesmo um método com aquela assinatura na superclasse. Como não existe `getDuracaoMinutos(String)` em `Atendimento`, a compilação falharia imediatamente e o bug morreria na hora de escrever, não em produção. É a diferença entre um erro que aparece em segundos e um que só aparece quando alguém percebe a duração errada na agenda. Por isso a anotação foi adicionada junto com a correção da assinatura.

### 5. Singleton manual vs bean do Spring (Aula 14)

O `GeradorProtocolo` garante que exista **uma única** instância na aplicação inteira, para que o contador de protocolos seja global e sequencial. O bug era que `getInstancia()` fazia `return new GeradorProtocolo()` sem guardar o objeto no campo estático `instancia`. Como o campo continuava `null`, toda chamada caía no `if` e criava um objeto novo, com `contador` zerado — daí a numeração 1, 1, 1 em vez de 1, 2, 3, e o `assertSame` falhando.

O `AgendaService` não corre esse risco porque quem controla o ciclo de vida dele é o Spring. A anotação `@Service` registra a classe como bean e o escopo padrão do container já é singleton: o Spring instancia uma vez, guarda no contexto e entrega sempre a mesma referência a quem pedir. A diferença é quem é responsável pela garantia — no Singleton manual, nós; no bean, o framework. Escrever o padrão à mão significa assumir também os detalhes que esquecemos aqui, como guardar a instância e proteger a criação em ambiente concorrente.

### 6. Cobertura de testes: onde parar? (Aula 15)

Vale a pena manter os que ficaram verdes. O `teste01` (consulta a R$ 150 fixo) não achou bug nenhum, mas congela uma decisão de negócio que é fácil de quebrar sem perceber — basta alguém "uniformizar" o cálculo de preço por porte nas três subclasses. Teste verde não é teste inútil: ele deixa de ser detector de bug e passa a ser rede de proteção contra regressão, que é exatamente o papel que os 20 testes entregues cumpriram quando corrigimos um bug e rodamos a suíte para ver se outro tinha quebrado.

Com prazo curto, priorizaríamos os caminhos de erro antes de perseguir cobertura total. O caminho feliz costuma ser exercitado à mão durante o desenvolvimento e quebra de forma barulhenta; os caminhos de erro são os que ninguém testa e que falham em silêncio — foi o caso dos quatro bugs que só apareceram quando escrevemos os testes que faltavam. Perseguir 100% de cobertura como número tem o efeito perverso de premiar testes triviais de getter; preferimos cobrir cada regra da tabela de contrato pelo menos uma vez, que é uma métrica ligada ao negócio e não à sintaxe.

---
