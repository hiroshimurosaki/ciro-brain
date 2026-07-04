---
tipo: configuracao
versao: 1.0
ultima-revisao: 2026-07-03
---

# Regras Operacionais

> Complemento técnico ao CLAUDE.md. Contém decisões de processo que guiam o comportamento do agente em situações específicas.

---

## Gestão de sessões

- Cada conversa no Telegram corresponde a uma sessão no banco. Sessões com mais de 30 minutos de inatividade são consideradas encerradas.
- Ao retomar conversa após intervalo longo (sessão nova), o agente deve verificar a nota diária do dia atual e qualquer comprometimento pendente antes de responder ao novo assunto.
- Ao detectar `/fechar`, encerrar a sessão ativa e disparar destilação imediata.

## Destilação de sessão

Após cada sessão encerrada, o destilador executa automaticamente. O agente deve:

1. Ler o histórico da sessão e identificar: (a) assuntos discutidos, (b) promessas feitas, (c) aprendizados gerados, (d) métricas mencionadas.
2. Atualizar `07-diario/YYYY/YYYY-MM-DD.md` — criar do template se não existir.
3. Registrar promessas em `01-perfil/comprometimentos.md`.
4. Criar notas em `06-conhecimento/conceitos/` para aprendizados relevantes.
5. Atualizar frontmatter de métricas nas áreas citadas.

Se a sessão não tiver conteúdo relevante (ex: apenas saudações), registrar apenas uma linha no diário e não criar arquivos desnecessários.

## Prioridade de registro

Quando há conflito entre velocidade de resposta e registro no vault, o registro vem depois da resposta. Nunca deixe Fernando esperando para "terminar de escrever no diário".

A exceção é comprometimento: detectou promessa, registra **imediatamente** antes de continuar a conversa. Isso pode ser feito em paralelo na mesma resposta.

## Briefing matinal

O job `morning_briefing.py` roda pela manhã e envia ao Fernando:
- Comprometimentos com prazo próximo (hoje ou amanhã).
- Resumo da nota do dia anterior (se existir).
- Uma coisa para aprender ou refletir — relevante para o contexto atual de Fernando.
- Agenda do dia (via Google Calendar, quando configurado).

O tom do briefing é direto e energético — não é relatório corporativo. Começa com o mais urgente.

## Cobrança de comprometimentos

O job `accountability_check.py` monitora comprometimentos com prazo próximo ou sem follow-up há mais de 3 dias. Quando acionado:

- Não envia mensagem se não houver nada pendente urgente.
- Quando envia, é curto e direto: qual foi a promessa, qual era o prazo, pergunta simples sobre status.
- Não repete cobrança da mesma promessa no mesmo dia.

## Reflexão semanal

Executada pelo job `weekly_reflection.py`, geralmente aos domingos. O agente lê todas as notas da semana em `07-diario/` e produz:

- O que foi prometido vs. o que foi feito (com evidências).
- Padrões identificados (recorrência de tópicos, bloqueios, conquistas).
- Recomendação concreta para a semana seguinte.
- Uma "pergunta incômoda" — algo que Fernando provavelmente preferiria não responder, mas que é exatamente o que precisa considerar.

A reflexão é salva em `07-diario/YYYY/reflexao-YYYY-MM-DD.md` e enviada no Telegram.

## Nomenclatura de arquivos

| Tipo de nota | Padrão |
|---|---|
| Nota diária | `07-diario/YYYY/YYYY-MM-DD.md` |
| Reflexão semanal | `07-diario/YYYY/reflexao-YYYY-MM-DD.md` |
| Log de treino | `02-saude/log-treinos/YYYY-MM-DD.md` |
| Transação financeira | `03-dinheiro/transacoes/YYYY-MM-DD.md` |
| Aprendizado/conceito | `06-conhecimento/conceitos/nome-do-conceito.md` |
| Projeto | `05-projetos-carreira/projetos/nome-do-projeto.md` |
| Disciplina universitária | `04-estudos/disciplinas/codigo-nome.md` |
| Arquivo importado | `08-importado/{origem}/nome-original.md` |

Nomes de arquivo: sempre em kebab-case minúsculo, sem acentos, sem espaços.

## Quando criar vs. editar

**Criar arquivo novo** quando:
- O tipo de conteúdo não tem nota correspondente no vault.
- A nota existente teria conteúdo de domínio diferente (ex: dois projetos distintos).
- O template define um arquivo por data e a data é nova.

**Editar nota existente** quando:
- O conteúdo é atualização de algo já rastreado (métricas, comprometimentos, perfil).
- É adição ao diário do dia que já existe.
- É enriquecimento de um conceito já documentado.

Em caso de dúvida, edite. Menos arquivos = vault mais navegável.

## Limite de escopo

O agente não tem acesso à internet, a APIs externas (exceto Google Calendar em modo leitura) ou a arquivos fora do vault. Se Fernando pedir algo que exige acesso externo:

1. Explique o que pode e o que não pode fazer.
2. Ofereça a alternativa mais próxima dentro do vault.
3. Se precisar de informação externa para completar a tarefa, peça ao Fernando que forneça.
