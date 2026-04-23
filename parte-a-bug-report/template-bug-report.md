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
