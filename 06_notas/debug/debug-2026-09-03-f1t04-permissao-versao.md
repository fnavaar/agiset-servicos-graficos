# Debug Summary — F1-T04

**Task e problema:** F1-T04 — cliente relatou que "conta comercial consegue ativar e retirar; todas as contas conseguem; não fiz a dupla ativação".

**Reprodução:**
- API com token da conta `comercial@homologacao.com`: tentativa de ATIVAR versão aprovada → **HTTP 403** "Somente o papel publicador pode ativar ou retirar".
- API com token da conta comercial: tentativa de RETIRAR versão → **HTTP 403**.
- Navegador logado como comercial: botão "Ativar (exige publicador)" / "Retirar (exige publicador)" aparece **desabilitado**.
- Dupla ativação via API com publicador (V-A ativa + tentar V-B): **HTTP 409** "Já existe uma versão ativa…".

**Causa raiz:**
- **Não há falha de permissão no backend nem no frontend atual.** O backend sempre bloqueou a conta comercial (403) e o frontend desabilita os botões para não-publicador.
- Hipóteses para o relato: (1) cliente testou antes da última correção do hook (v0.0.19), quando `e.record` no request hook já trazia o body aplicado e a lógica de validação de estado estava incorreta; (2) cliente usou a conta publicador nos dois primeiros passos (criação de coorte/versão, permitida para qualquer conta autenticada) e depois permaneceu na mesma sessão.
- Durante o debug, foi identificado e corrigido um bug real no hook: `e.record` no `onRecordUpdateRequest` já contém o estado NOVO (body aplicado). A comparação com estado antigo falhava. Corrigido lendo o estado persistido via `$app.findRecordById` (v0.0.19).

**Correção aplicada:** hook `versao_grade_unica.js` reescrito — lê `estadoAntigo` do banco persistido, valida permissão de publicador (403), dupla ativação (409) e grava auditoria em `e.record` (publicado_por/em, retirado_por/em, histórico).

**Verificação automática:** GREEN (publicador ativa/retira com auditoria), RED (dupla ativação 409 conflito), 403 para comercial ativar/retirar, 403 para autopromoção de papel. QA pipeline v0.0.19 verde.

**Gate atual:** aguardando teste humano (re-teste do cliente no preview).