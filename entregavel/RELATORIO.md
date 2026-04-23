# 📦 Relatório Final

> **Atividade:** Bug Report Profissional + Code Review Guiado
> **Curso:** Qualidade de Software
> **Professor:** Prof. Claudio Nunes

---

## 👥 Identificação da dupla

| Nome completo | RA | GitHub |
|---|---|---|
| [Luan Robert Santos de Araújo] | [232750] | [@Luan-Robert] |
| [Matheus da Silva Maia] | [229951] | [@MaiaMayan] |

**Ambiente de testes:** [Descreva brevemente o setup — ex: Chrome 121 no Windows 11, GitHub Pages do fork, editor web do GitHub]

---

## 📋 Sumário

- [Parte A — Bug Reports](#parte-a--bug-reports)
- [Parte B — Code Review](#parte-b--code-review)
- [Reflexão final](#-reflexão-final)
- [Declarações](#-declarações)

---

## Parte A — Bug Reports

# 🐛 Bug Reports — Parte A

**Dupla:** [Luan Robert 232750] + [Matheus Maia 229951]
**Data da exploração:** [22/04/2026]
**Navegador usado:** [Chrome 121 / Firefox 122 / Safari 17 / …]
**Sistema operacional:** [Windows 11 / macOS 14 / Ubuntu 22.04 / …]

---

Aqui está o relatório de bug preenchido com base no comportamento que você descreveu, focado em clareza e com informações técnicas úteis para quem for atuar na correção:

-----

## BUG-001

**Título:** [Formulário de Tarefa] Campo "Prioridade (1-5)" permite a inserção e salvamento de valores fora do limite (negativos e \> 5)

**Severidade:** Média
**Justificativa da severidade:** A falta de validação permite a inserção de dados inconsistentes no banco de dados. Embora não cause a quebra imediata ou travamento da aplicação, compromete a integridade dos dados e pode quebrar lógicas de negócio ou ordenação que dependam estritamente de valores entre 1 e 5.

**Prioridade:** P2
**Justificativa da prioridade:** Trata-se de um vazamento de regra de negócio em uma funcionalidade de uso contínuo. Deve ser corrigido na próxima iteração/sprint para evitar a poluição do banco de dados com dados sujos e garantir a confiabilidade da listagem de tarefas.

**Ambiente:**

  - Navegador: Chrome 121.0
  - Sistema Operacional: Windows 11
  - Versão da aplicação: TarefaQS v1.0.0

**Passos para reprodução:**

1.  Acessar a aplicação e navegar até a tela de criação ou edição de uma tarefa.
2.  Localizar o campo numérico de "Prioridade (1-5)".
3.  Digitar um valor negativo (ex: `-1`) ou um valor superior a 5 (ex: `6`).
4.  Preencher os demais campos obrigatórios, se houver.
5.  Clicar no botão para salvar a tarefa.

**Resultado esperado:**
O sistema deve impedir a submissão do formulário. O front-end deve exibir uma mensagem de erro clara informando que o valor deve estar entre 1 e 5, e a API deve rejeitar a requisição retornando um erro `400 Bad Request` caso o dado chegue até ela.

**Resultado obtido:**
O formulário é submetido sem interrupções e a API aceita e persiste a tarefa com a prioridade inválida informada pelo usuário.

**Evidência:**

> Se preferir anexar um GIF ou arquivo de log, crie uma pasta `evidencias/` ao lado deste arquivo e referencie o arquivo aqui.

**Sugestão de causa raiz (opcional):**
Ausência de validação dupla (front-end e back-end).
No HTML, a tag `<input type="number">` provavelmente está sem os atributos de limite `min="1"` e `max="5"`. No back-end, faltam regras de validação no fluxo de entrada. Recomenda-se implementar a validação diretamente no Endpoint ou no Handler da requisição (utilizando ferramentas como o FluentValidation, por exemplo) para garantir que a entidade só chegue ao Entity Framework com o estado perfeitamente válido.

---

BUG-002
Título: [Formulário de Tarefa] Campo de data permite a inserção de ano com 6 dígitos (ex: 275760)

Severidade: Alta
Justificativa da severidade: Datas com anos muito além do limite padrão (geralmente o ano 9999 na maioria dos bancos de dados relacionais) costumam gerar exceções não tratadas quando o sistema tenta persistir a informação. Isso pode resultar em um Erro 500 (Internal Server Error) na API, quebra do fluxo de salvamento ou corrupção na lógica de ordenação e exibição no front-end caso o valor seja salvo.

Prioridade: P2
Justificativa da prioridade: Embora o usuário não digite esse ano em um cenário normal, erros de digitação ocorrem com frequência. A aplicação deve ser resiliente a isso, informando o erro amigavelmente em vez de tentar processar um dado impossível e gerar falhas técnicas.

Ambiente:

Navegador: Chrome 121.0

Sistema Operacional: Windows 11

Versão da aplicação: TarefaQS v1.0.0

Passos para reprodução:

Acessar a aplicação e abrir a tela de criação ou edição de uma tarefa.

Localizar o campo correspondente à data (ex: Data de Vencimento / Data de Criação).

Preencher o campo de data digitando manualmente o valor "09/09/275760".

Preencher os demais campos obrigatórios.

Clicar em salvar.

Resultado esperado:
O front-end deve aplicar uma máscara ou limite que impeça a digitação de anos com mais de 4 dígitos (ex: limitando ao ano 9999). A API deve validar a requisição e retornar um 400 Bad Request informando que a data está fora de um intervalo razoável (ex: "A data informada é inválida").

Resultado obtido:
O sistema permite a digitação livre de 6 dígitos no ano, o formulário é submetido e a aplicação tenta processar a data irreal.

Evidência:

Sugestão de causa raiz (opcional):
O problema se divide em duas camadas. No front-end, o campo deve estar permitindo digitação livre sem uma máscara estrita (mask) ou sem os atributos min e max adequados. No back-end, o bind do modelo provavelmente está convertendo o valor gigantesco, o que causará um estouro de limite ao tentar mapear a entidade para o banco de dados via Entity Framework Core (já que tipos como DATETIME no SQL Server suportam apenas até 31/12/9999). É necessário adicionar uma regra de validação no fluxo de entrada (como no Handler da operação) para garantir que a data pertença a um range lógico de negócio (ex: entre o ano 2000 e 2100).

---

BUG-003
Título: [Dashboard] Contador de tarefas "Pendentes" duplica a contagem de itens com Prioridade 5

Severidade: Média
Justificativa da severidade: Apresenta dados inconsistentes e matematicamente errados ao usuário. Embora não impeça a criação de tarefas, compromete a credibilidade da dashboard e das métricas de produtividade da aplicação.

Prioridade: P3
Justificativa da prioridade: É um erro de lógica na exibição de informações. Deve ser corrigido para garantir que a soma de "Pendentes + Concluídas" seja sempre igual ao "Total".

Ambiente:

Navegador: Chrome 121.0

Sistema Operacional: Windows 11

Versão da aplicação: TarefaQS v1.0.0

Passos para reprodução:

Criar uma nova tarefa com qualquer título e definir a Prioridade como 5.

Manter a tarefa como Pendente (não marcar o checkbox).

Observar o card de resumo "PENDENTES" no topo da tela.

Notar que, para cada tarefa de prioridade 5, o contador incrementa 2 unidades.

Resultado esperado:
O contador de "PENDENTES" deve exibir apenas a quantidade real de tarefas não concluídas (no caso da evidência, o valor correto seria 1).

Resultado obtido:
O contador exibe 2 tarefas pendentes, pois a tarefa "Maia" (Prioridade 5) está sendo contabilizada duas vezes.

Evidência:

Sugestão de causa raiz (opcional):
Provavelmente há um erro na query ou na função de agregação (Javascript no front ou LINQ no back-end). É possível que a lógica de contagem esteja usando um OR inadequado ou dois métodos Sum/Count que se sobrepõem, algo como:
contar onde (status == pendente) + contar onde (prioridade == 5).
O correto seria filtrar estritamente pelo status de conclusão, independente do peso da prioridade.

---

## Parte B — Code Review

# 🔎 Formulário — Parte B

**Dupla:** [Luan Robert 232750] + [Matheus Maia 229951]  
**Data da revisão:** 22/04/2026

---

### Finding #1

**📍 Linha(s):** 5 e 18  
**🏷 Rótulo:** `nit`  
**📂 Dimensão:** Padrões  
**⚠️ Severidade:** Baixa  

**🐛 Problema:** A constante `TIPOS_VALIDOS` foi declarada e exportada, mas não é utilizada internamente para validar o campo `tipo` durante o cadastro de novos usuários. Isso permite que qualquer string seja salva no banco, ignorando a regra de negócio.

**💡 Sugestão de correção:**
Adicionar uma verificação no início da função `cadastrarUsuario`.

```javascript
// ajuste sugerido
if (!TIPOS_VALIDOS.includes(dados.tipo)) {
    throw new Error('Tipo de usuário inválido');
}
```

---

### Finding #2

**📍 Linha(s):** 7, 11, 16 e 32  
**🏷 Rótulo:** `major`  
**📂 Dimensão:** Tratamento de erros  
**⚠️ Severidade:** Média  

**🐛 Problema:** As funções assíncronas que interagem com o banco de dados (`db.executarQuery`, `db.insert`, etc.) não possuem blocos `try/catch`. Se o banco estiver offline ou a query falhar, a aplicação irá sofrer um *unhandled promise rejection*, o que pode derrubar o processo do Node.js.

**💡 Sugestão de correção:**
Envolver as chamadas em blocos de tratamento de erro.

```javascript
// exemplo na listarUsuariosAtivos
async function listarUsuariosAtivos() {
  try {
    return await db.executarQuery('SELECT * FROM usuarios WHERE ativo = 1');
  } catch (error) {
    logger.error('Erro ao listar usuários: ' + error.message);
    throw error;
  }
}
```

---

### Finding #3

**📍 Linha(s):** 12  
**🏷 Rótulo:** `blocker`  
**📂 Dimensão:** Segurança  
**⚠️ Severidade:** Crítica  

**🐛 Problema:** **SQL Injection.** A variável `nome` é concatenada diretamente na string da query. Um atacante poderia enviar algo como `' OR '1'='1` para expor todos os dados da tabela ou comandos maliciosos para deletar registros.

**💡 Sugestão de correção:**
Utilizar *Prepared Statements* ou parâmetros fornecidos pela biblioteca de banco de dados.

```javascript
// ajuste sugerido
async function buscarUsuarioPorNome(nome) {
  const query = "SELECT * FROM usuarios WHERE nome = ?";
  return db.executarQuery(query, [nome]);
}
```

---

### Finding #4

**📍 Linha(s):** 41 a 100  
**🏷 Rótulo:** `major`  
**📂 Dimensão:** Complexidade  
**⚠️ Severidade:** Alta  

**🐛 Problema:** A função `calcularLimiteEmprestimo` possui uma **complexidade ciclomática extremamente alta** devido ao aninhamento excessivo de `if/else` (o famoso "código hadouken"). Isso torna a manutenção difícil e aumenta o risco de bugs em modificações futuras.

**💡 Sugestão de correção:**
Utilizar a técnica de **Guard Clauses** (cláusulas de guarda) para retornar valores antecipadamente ou extrair a lógica para uma estrutura de dicionário/estratégia.

```javascript
// exemplo de simplificação com Guard Clause
if (usuario.bloqueadoAte && new Date(usuario.bloqueadoAte) > hoje) return 0;
if (usuario.tipo === 'visitante') return 5; // valor padrão
// seguir com lógicas menores por tipo...
```

---

### Finding #5

**📍 Linha(s):** 41 a 141  
**🏷 Rótulo:** `major`  
**📂 Dimensão:** Complexidade  
**⚠️ Severidade:** Média  

**🐛 Problema:** **Violação do princípio DRY (Don't Repeat Yourself).** As funções `calcularLimiteEmprestimo` e `calcularLimiteComSuspensao` compartilham quase 90% da mesma lógica de cálculo de limites por tipo e atrasos, duplicando o esforço de manutenção.

**💡 Sugestão de correção:**
Unificar o cálculo em uma única função que receba parâmetros opcionais ou tratar a suspensão como uma regra de guarda no início de uma função única.

```javascript
// ajuste sugerido
function calcularLimite(usuario) {
    if (usuario.suspenso || estaBloqueado(usuario)) return 0;
    // ... lógica unificada aqui
}
```

---

### Finding #6

**📍 Linha(s):** 33  
**🏷 Rótulo:** `nit`  
**📂 Dimensão:** Legibilidade  
**⚠️ Severidade:** Baixa  

**🐛 Problema:** O uso da variável `u` é genérico e pouco descritivo. Em um arquivo maior ou em funções mais complexas, nomes de uma única letra dificultam a leitura e o entendimento do contexto por outros desenvolvedores.

**💡 Sugestão de correção:**
Renomear a variável para algo semanticamente correto, como `usuario` ou `usuarioEncontrado`.

```javascript
// ajuste sugerido
const usuario = await db.buscarPorId('usuarios', id);
usuario.email = novoEmail;
```

---

## ✅ Checklist final

- [x] Há pelo menos 6 findings preenchidas
- [x] Cada finding cita linha, dimensão, rótulo e severidade
- [x] As sugestões são concretas e acionáveis
- [x] Pelo menos uma finding cobre segurança (SQL Injection)
- [x] Pelo menos uma finding cobre complexidade (Nested Ifs)

### Resumo

Aqui está o resumo consolidado e as findings detalhadas para o seu relatório final. Com esta tabela, quem ler o documento consegue bater o olho e entender logo de cara onde estão os maiores riscos (especialmente aquele SQL Injection na linha 12).

---

### Resumo

| # | Linha | Dimensão | Rótulo | Severidade |
|---|-------|----------|--------|------------|
| 1 | 5, 18 | Padrões | `nit` | Baixa |
| 2 | 7, 11, 16, 32 | Erros | `major` | Média |
| 3 | 12 | Segurança | `blocker` | Crítica |
| 4 | 41-100 | Complexidade | `major` | Alta |
| 5 | 41-141 | Complexidade | `major` | Média |
| 6 | 33 | Legibilidade | `nit` | Baixa |

---

### Findings detalhadas

### Finding #1

**📍 Linha(s):** 5 e 18
**🏷 Rótulo:** `nit`
**📂 Dimensão:** Padrões
**⚠️ Severidade:** Baixa

**🐛 Problema:** A constante `TIPOS_VALIDOS` foi declarada e exportada, mas não é utilizada internamente para validar o campo `tipo` durante o cadastro de novos usuários. Isso permite que qualquer string seja salva no banco, ignorando a regra de negócio.

**💡 Sugestão de correção:**
Adicionar uma verificação no início da função `cadastrarUsuario`.

```javascript
if (!TIPOS_VALIDOS.includes(dados.tipo)) {
    throw new Error('Tipo de usuário inválido');
}
```

---

### Finding #2

**📍 Linha(s):** 7, 11, 16 e 32
**🏷 Rótulo:** `major`
**📂 Dimensão:** Erros
**⚠️ Severidade:** Média

**🐛 Problema:** As funções assíncronas que interagem com o banco de dados (`db.executarQuery`, `db.insert`, etc.) não possuem blocos `try/catch`. Se o banco estiver offline ou a query falhar, a aplicação irá sofrer um *unhandled promise rejection*, o que pode interromper o processo do Node.js.

**💡 Sugestão de correção:**
Envolver as chamadas em blocos de tratamento de erro para logar a falha adequadamente.

```javascript
async function listarUsuariosAtivos() {
  try {
    return await db.executarQuery('SELECT * FROM usuarios WHERE ativo = 1');
  } catch (error) {
    logger.error('Erro ao listar usuários: ' + error.message);
    throw error;
  }
}
```

---

### Finding #3

**📍 Linha(s):** 12
**🏷 Rótulo:** `blocker`
**📂 Dimensão:** Segurança
**⚠️ Severidade:** Crítica

**🐛 Problema:** **SQL Injection.** A variável `nome` é concatenada diretamente na string da query. Um atacante pode enviar comandos maliciosos para extrair dados sensíveis ou deletar tabelas inteiras.

**💡 Sugestão de correção:**
Utilizar *Prepared Statements* (parâmetros) em vez de concatenação de strings.

```javascript
async function buscarUsuarioPorNome(nome) {
  const query = "SELECT * FROM usuarios WHERE nome = ?";
  return db.executarQuery(query, [nome]);
}
```

---

### Finding #4

**📍 Linha(s):** 41 a 100
**🏷 Rótulo:** `major`
**📂 Dimensão:** Complexidade
**⚠️ Severidade:** Alta

**🐛 Problema:** A função `calcularLimiteEmprestimo` apresenta uma complexidade ciclomática excessiva. O aninhamento profundo de `if/else` dificulta o teste unitário e a compreensão da lógica de negócio.

**💡 Sugestão de correção:**
Refatorar utilizando **Guard Clauses** para reduzir o aninhamento.

```javascript
if (usuario.bloqueadoAte && new Date(usuario.bloqueadoAte) > hoje) return 0;
if (usuario.tipo === 'professor') {
    // lógica simplificada...
}
```

---

### Finding #5

**📍 Linha(s):** 41 a 141
**🏷 Rótulo:** `major`
**📂 Dimensão:** Complexidade
**⚠️ Severidade:** Média

**🐛 Problema:** **Código Duplicado (Violação do DRY).** As funções `calcularLimiteEmprestimo` e `calcularLimiteComSuspensao` repetem quase a mesma lógica. Qualquer mudança futura em uma regra de negócio precisará ser replicada em dois lugares diferentes.

**💡 Sugestão de correção:**
Extrair a lógica de cálculo para uma única função reutilizável que trate a condição de suspensão como um filtro inicial.

```javascript
function calcularLimiteUnificado(usuario) {
    if (usuario.suspenso) return 0;
    // ... restante da lógica
}
```

---

### Finding #6

**📍 Linha(s):** 33
**🏷 Rótulo:** `nit`
**📂 Dimensão:** Legibilidade
**⚠️ Severidade:** Baixa

**🐛 Problema:** Nome de variável pouco descritivo (`u`). Variáveis de uma única letra dificultam o rastreamento do código, especialmente em revisões rápidas ou manutenção por outros membros da equipe.

**💡 Sugestão de correção:**
Alterar para um nome que represente a entidade.

```javascript
const usuarioEncontrado = await db.buscarPorId('usuarios', id);
usuarioEncontrado.email = novoEmail;
```

---

## 💭 Reflexão final

> Responda em 1-2 parágrafos. Esta reflexão **é obrigatória**.

**Qual dimensão do checklist foi mais difícil aplicar? Por quê?**

Segurança com o SQLINJECTION que era dificil compreender como seria o codigo onde é possivel fazer o slqinjection, afinal apredemos a maneira correta, mas nunca a errada "como fazer", ou como fica aparencia da errada

**O que vocês fariam diferente se revisassem o código novamente?**

o aprendizado dos erros e as informações que adquirimos iria ajudar a encontrar os erros mais rápido e com mais facilidade, pois algo teriamos experiencia previa identificando.

---

## 📣 Declarações


### Uso de IA como parceiro de trabalho

- [ ] Não usamos IA nesta atividade.
- [X] Usamos IA para esclarecer conceitos teóricos.
- [X] Usamos IA para revisar a redação dos bug reports.
- [X] Usamos IA para discutir se um achado era ou não um defeito.
- [X] Uso específico: [usamos IA para escrever o texto sobre os erros que já haviamos achado]

### Declaração de autoria

Declaramos que este relatório é de autoria da dupla, que exploramos
pessoalmente a aplicação da Parte A e lemos o código da Parte B. As
findings aqui registradas representam nosso próprio julgamento
técnico.
