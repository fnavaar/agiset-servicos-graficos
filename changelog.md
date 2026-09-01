# Changelog — Projeto Agiset Serviços Gráficos

> Registro de tudo que acontece no projeto, em ordem cronológica inversa (mais recente no topo).
> Formato: `- data · [quem] · o que aconteceu`
> **Dúvidas para o consultor** entram como: `- data · [quem] · DÚVIDA: …` — ele responde
> na próxima sincronização.

## Registro

- 2026-09-01 · [Felippe] · Task F1-T01 concluída: teste humano aprovado (cadastro salvo, código imutável confirmado). Verificação reexecutada do zero: criação empresa/contato/solicitação OK via API autenticada; edição do `solicitacao_codigo` bloqueada por hook (registro manteve `SOL-111222`); sem campos de preço/cotação; empresa Agiset confirmada no banco. Hook `solicitacao_codigo_imutavel.js` adicionado (v0.0.10). Aprendizado registrado em `06_notas/aprendizado-continuo/AP-2026-09-01-1535-imutabilidade-hook.md`.
- 2026-09-01 · [Francisneverson / SkillMind] · F1-T01 debug: teste humano falhou (cadastro não salvo). Causa raiz: rules exigem `@request.auth.id` mas frontend não tinha autenticação → POST empresa sem token retornava 400. Corrigido: tela Login/Registro (`src/pages/Login.tsx`), rota protegida (`RequireAuth`), logout no layout. v0.0.9 QA green. Usuário debug removido.
- 2026-08-21 · [Francisneverson / SkillMind] · F1-T01 seed: empresa Agiset Serviços Gráficos cadastrada no banco com dados reais (CNPJ 15.651.010/0001-64, IE 206.163.153.115, endereço Estrada da Balsa 888 Barueri/SP, tel 11 2543 4300, site www.agiset.com). Migration 0003. v0.0.7 QA green.
- 2026-08-21 · [Francisneverson / SkillMind] · F1-T01 ajuste: Migration 0002 atualizou schema com campos da planilha Dados_Cadastros — empresas (CNPJ, IE, nome_fantasia, segmento select, endereco, cep, estado, local_entrega, site, instagram, telefone), contatos (area/departamento), solicitacoes (origens: indicacao, google, instagram, facebook, email, anuncio, outros). Frontend atualizado. v0.0.6.
- 2026-08-21 · [Francisneverson / SkillMind] · F1-T01 implementada: migration 0001 criou collections `empresas`, `contatos` e `solicitacoes` no PocketBase (Skip Cloud); frontend com formulário de nova solicitação e lista; código imutável `SOL-XXXXXX` gerado no frontend; estados `rascunho`, `aguardando_dados`, `em_excecao_humana` configurados; QA pipeline green (v0.0.2→v0.0.4).
- 2026-08-21 · [Consultoria Adapta] · Repositório público criado e publicado em https://github.com/fnavaar/agiset-servicos-graficos; contém somente o handoff validado da Fase 1 para execução controlada.