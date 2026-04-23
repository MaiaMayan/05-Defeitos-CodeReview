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
