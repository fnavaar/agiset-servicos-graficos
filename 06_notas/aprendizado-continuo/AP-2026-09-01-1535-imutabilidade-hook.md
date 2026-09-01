# AP-2026-09-01-1535-imutabilidade-hook.md — Identificador imutável exige hook no Skip Cloud

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T01 / SPEC-F1-001 (CA-1-01)
- Sinal: `autogeneratePattern` no PocketBase gera valor mas NÃO impede edição via API; PATCH em `solicitacao_codigo` alterava o valor. Após hook `onRecordUpdateRequest` bloqueando o campo, PATCH retorna sem alterar o registro.
- Evidência: teste curl — PATCH `solicitacao_codigo` → registro manteve `SOL-111222` (valor original); edição de outro campo (`estado`) funcionou normalmente.
- Regra reutilizável: em Skip Cloud, para tornar um campo imutável, usar `onRecordUpdateRequest` (hooks) verificando `e.collection.name` e bloqueando com `e.badRequestError`/interrompendo a cadeia — `autogeneratePattern` sozinho não garante imutabilidade.
- Quando aplicar: qualquer campo identificador que não deva ser editado (código, ID de negócio).
- Quando não aplicar: campos editáveis por regra de negócio.
- Confiança: alta — reproduzido com evidência observável (PATCH 200 sem mudança no valor).
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.