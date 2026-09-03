# Estado atual — Adapta Cliente

- task_id: F1-T04
- champion: Felippe
- spec: 04_fase-atual/specs/spec-f1-002-grade-e-publicacao-controlada.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada — 2026-09-01T18:26:00-03:00 — "ok continue e me diga o que fazer"
- teste_humano: pendente — aguardando Felippe testar dupla ativação e permissão da conta comercial no preview (backend já provado: 409 dupla ativação, 403 comercial)
- verificacao_automatica: passou — GREEN (publicador ativa/retira), RED (dupla ativação 409 conflito), 403 para usuário comum ativar/retirar e para autopromoção; v0.0.19 QA green
- aprendizado: pendente
- ultima_acao: F1-T04 implementada e debugada — causa do relato do cliente: backend já bloqueava (403), frontend já desabilitava; ambiente de teste recriado (coorte "Coorte Demonstração", V-A ativa, V-B aprovada) para re-teste limpo
- proxima_acao: Felippe testa dupla ativação (V-B com V-A ativa → erro) e tenta ativar/retirar com conta comercial (deve bloquear)
- atualizado_em: 2026-09-03T10:40:00-03:00