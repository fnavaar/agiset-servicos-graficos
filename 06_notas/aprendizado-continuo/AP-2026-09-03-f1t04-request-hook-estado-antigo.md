# AP-2026-09-03-f1t04-request-hook-estado-antigo — No request hook do PocketBase, e.record já tem o body aplicado

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-T04 (F1-002 — RN-05, CA-1-07)
- Sinal: no `onRecordUpdateRequest` do PocketBase (Skip Cloud), `e.record` **já contém os valores do body aplicados** (estado NOVO). Validar transições usando `record.getString('estado')` dentro do request hook compara sempre com o valor novo, quebrando regras baseadas no estado anterior (ex.: "só aprovada → ativa" falhava e bloqueava tudo com 400).
- Evidência: debug F1-T04 — ativação sempre retornava 400 "Somente uma versão aprovada pode ser ativada" mesmo com o registro aprovado; `console.log` no hook mostrou `body.estado=ativa record.estado=ativa`. Correção: ler o estado persistido ANTES do update via `$app.findRecordById('versoes_grade', e.record.id)` e validar contra esse valor antigo (v0.0.19) — ativação passou a funcionar e dupla ativação passou a dar 409 correto.
- Regra reutilizável: em request hooks de update que precisam do valor ANTERIOR de um campo, use `$app.findRecordById` (ou similar) para ler o registro persistido; nunca confie em `e.record` para o "estado antes".
- Quando aplicar: validação de transições de estado, imutabilidade e regras que dependem do valor pré-update em hooks `onRecordUpdateRequest`/`onRecordBeforeUpdateRequest`.
- Quando não aplicar: hooks de create (não há registro anterior), ou quando a regra valida apenas o valor novo enviado.
- Confiança: alta — reproduzido de forma determinística; correção verificada no fluxo real.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.