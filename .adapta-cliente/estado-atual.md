# Estado atual — Adapta Cliente

- task_id: F1-T04
- champion: Felippe
- spec: 04_fase-atual/specs/spec-f1-002-grade-e-publicacao-controlada.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada — 2026-09-01T18:26:00-03:00 — "ok continue e me diga o que fazer" (após relatório de análise da F1-T04)
- teste_humano: pendente — aguardando Felippe testar coortes/versões/papel de publicador no preview
- verificacao_automatica: passou — GREEN (publicador ativa V-A com auditoria), RED (dupla ativação 409 conflito), 403 para usuário comum ativar/retirar e para autopromoção de papel; v0.0.19 QA green
- aprendizado: pendente
- ultima_acao: F1-T04 implementada — migration 0005 (coortes, versoes_grade, itens_elegiveis + campo papel), migration 0006 (seed contas homologação), hook versao_grade_unica.js, páginas Coortes e VersoesGrade; dados de teste limpos
- proxima_acao: Felippe testa — criar coorte, versão V-A/V-B, aprovar, ativar, tentar dupla ativação e permissões
- atualizado_em: 2026-09-01T18:37:00-03:00