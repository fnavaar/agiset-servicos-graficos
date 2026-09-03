# Changelog — Projeto Agiset Serviços Gráficos

> Registro de tudo que acontece no projeto, em ordem cronológica inversa (mais recente no topo).
> Formato: `- data · [quem] · o que aconteceu`
> **Dúvidas para o consultor** entram como: `- data · [quem] · DÚVIDA: …` — ele responde
> na próxima sincronização.

## Registro

- 2026-09-03 · [Fran (champion)] · Task F1-T04 concluída ("pode fechar" após teste automatizado no navegador): coortes, versões de grade, papel de publicador. Evidências: publicador ativa/retira com auditoria (retirado_por/em e histórico gravados); dupla ativação bloqueada (409 "Já existe uma versão ativa"); comercial bloqueada na UI (botão desabilitado) e na API (403 em ativar e retirar); autopromoção de papel bloqueada (403). Debug prévio: relato de permissão era de versão anterior do hook (e.record com body aplicado); corrigido lendo estado persistido via `$app.findRecordById` (v0.0.19 QA green). Contas de homologação criadas (publicador/comercial). Aprendizado registrado em `06_notas/aprendizado-continuo/AP-2026-09-03-f1t04-request-hook-estado-antigo.md`.
- 2026-09-03 · [Felippe] · DEBUG task F1-T04: relato "conta comercial consegue ativar/retirar" → verificado via API: backend bloqueia comercial (403) e frontend desabilita; dupla ativação bloqueada (409). Causa raiz do relato: teste possivelmente antes da correção do hook (v0.0.19) — onde `e.record` no request hook já trazia o body aplicado e a validação de estado estava incorreta; corrigido lendo estado persistido via `$app.findRecordById`. Ambiente recriado para re-teste. Debug Summary em `06_notas/debug/debug-2026-09-03-f1t04-permissao-versao.md`.
- 2026-09-01 · [Fran (champion)] · Task F1-T02 concluída: autorizou teste e conclusão ("teste voce e conclua"). Verificação do zero: cenário 1 (reentrada A/X) retorna 409 com `retomada:true` e mesmo `solicitacao_id` — sem duplicação; cenário 2 (contato X em empresa B) retorna 409 com `conflito:true` e abre revisão sem mesclar. Falha visual do teste anterior corrigida: frontend agora reconhece o corpo do hook (409) e exibe "Divergência de vínculo detectada" com redirecionamento para /revisoes (v0.0.13 QA green). Testado ponta a ponta no browser. Revisões/empresas/contatos de teste removidos. Aprendizado em `06_notas/aprendizado-continuo/`.
- 2026-09-01 · [Felippe] · Task F1-T01 concluída: teste humano aprovado. Verificação reexecutada: criação OK via API autenticada; `solicitacao_codigo` imutável por hook; sem preço/cotação; Agiset confirmada. v0.0.10. AP-2026-09-01-1535.
- 2026-09-01 · [Francisneverson / SkillMind] · F1-T01 debug: falta de auth no frontend → Login/Registro, RequireAuth, logout. v0.0.9.
- 2026-08-21 · [Francisneverson / SkillMind] · F1-T01 seed Agiset (dados reais). v0.0.7.
- 2026-08-21 · [Francisneverson / SkillMind] · F1-T01 ajuste de schema (campos da planilha). v0.0.6.
- 2026-08-21 · [Francisneverson / SkillMind] · F1-T01 implementada (migrations 0001-0002, formulário, lista, estados). v0.0.2→v0.0.4.
- 2026-08-21 · [Consultoria Adapta] · Repositório público criado.