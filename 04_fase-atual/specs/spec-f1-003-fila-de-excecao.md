# SPEC-F1-003 — Fila de exceção e encaminhamento sem compromisso automático

**Fase:** 1  
**Status:** planejada  
**Dono:** Comercial  
**Origem no escopo:** Fase 1; DC-01, DC-02 e DC-04; RQ-003, RQ-004 e RQ-009  
**Degrau da solução:** recurso nativo do portal — a fila usa os registros e estados já introduzidos na Fase 1, sem bot, conector ou motor de decisão adicional.

## Contexto e decisões fechadas

- **Estado atual:** demandas fora da linha podem voltar à conversa aberta e ao atendimento artesanal, com risco de promessa antes da regra aprovada.
- **Estado desejado:** toda falta de dado, item fora da grade ou divergência de vínculo produz uma pendência ou exceção com motivo, responsável, contexto e resultado terminal rastreável.
- **Decisões já fechadas:** exceção é humana; não cria preço, condição, proposta, OP ou alteração automática de grade; WhatsApp não é sistema de workflow.
- **Bloqueios:** nenhum para a SPEC. Alçadas, SLA, frete, desconto, pagamento e retorno a cotação são deliberadamente de fases posteriores; não devem ser configurados como regra nesta fase.

## Resultado observável

A Comercial demonstra uma fila centralizada onde uma solicitação fora da grade ou com dado faltante possui motivo e próxima ação. Ela registra uma decisão sem apagar o histórico e o portal impede qualquer mensagem, cotação ou compromisso automático.

## Limites e dependências

- **Inclui:** classificação de pendência/exceção, fila por estado e responsável, motivos mínimos, contexto de origem, atribuição/reatribuição, comentário operacional, decisão registrada e histórico.
- **Fora de escopo:** aprovação automática, negociação, desconto, preço, SLA obrigatório, notificações externas automáticas, bot de atendimento, alteração de catálogo, pedido/OP ou produção.
- **Entradas e pré-condições:** SPEC-F1-001 concluída em homologação; SPEC-F1-002 ou um dado obrigatório deve ser capaz de disparar exceção; contas de teste para Comercial e cliente/lead.
- **Saídas/artefatos:** fila `aguardando dados` e `em exceção humana`, registro de motivo, responsável, resultado e evidências de roteiros.
- **Dependências e responsáveis:** Comercial classifica e encerra; patrocinador define somente a coorte/grade; qualquer futura mudança de regra retorna a aprovação de versão na SPEC-F1-002, nunca é feita pela fila.
- **Atores e permissões mínimas:** cliente/lead vê a orientação de completar dados, sem assumir resultado; Comercial lê/escreve exceções atribuídas; patrocinador consulta. Cliente não altera classificação, dono nem decisão terminal.
- **Superfícies/arquivos/configurações afetadas:** estados e transições de solicitação, painel/fila de Comercial, campos `motivo`, `contexto`, `responsável`, `próxima_ação`, `decisão` e histórico. Nenhuma notificação externa.
- **Risco e plano B:** uma exceção esquecida pode recriar o atendimento informal; a Comercial consulta a fila no início e fim de cada turno da coorte e registra a ação no portal.
- **Rollback ou reversão:** desativar visualização de entrada pública se houver erro de configuração, preservar fila/histórico e continuar classificação assistida pela Comercial; nenhuma decisão terminal é apagada.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Solicitação → fila do portal | Portal | `solicitacao_id`, estado, motivo, contexto, responsável, próxima ação, timestamps, decisão e histórico | Comercial escreve; patrocinador consulta | Reabrir a mesma exceção conserva o histórico e não duplica o registro | Falta de responsável impede encerramento; permanece visível na fila |
| Comercial → cliente/lead | Portal | Orientação registrada no histórico, sem preço/prazo/condição | Apenas Comercial envia comunicação manual aprovada | Não há automação ou integração de mensagem | Falha de comunicação mantém pendência aberta e registrada |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-09 | Campo mínimo ausente | `aguardando dados`, motivo e próxima ação | Comercial completa com histórico | RQ-001 |
| RN-10 | Item ou versão não elegível | `em exceção humana` com contexto da escolha | Não alterar catálogo nem gerar cotação | RQ-004 |
| RN-11 | Exceção recebe decisão | Registrar no histórico uma orientação manual, recusa ou cancelamento com responsável; o estado continua `em exceção humana` até ser recusado/cancelado | Retorno a cotação só pode existir na Fase 2, por nova versão autorizada | RQ-004 |
| RN-12 | Responsável ausente ou indisponível | Manter item aberto e visível à Comercial | Não concluir por silêncio | DC-05 |

## Fluxo e regras

1. O portal identifica campo obrigatório ausente, item fora da grade ou divergência de vínculo e abre a pendência/exceção com contexto.
2. A Comercial assume ou atribui responsável, registra próxima ação e mantém o cliente sem preço, condição ou promessa.
3. A decisão fica registrada no histórico; somente recusa ou cancelamento encerra a solicitação. Nos demais casos, ela permanece em `em exceção humana` e continua consultável pelo identificador.
4. A fila não muda catálogo nem libera cotação. Qualquer sinal de nova regra vira insumo para aprovação futura da grade, fora desta fase.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Item fora da grade | Exceção com motivo, contexto, dona/o e próxima ação, mantida em `em exceção humana` | Comercial informa que depende de avaliação humana |
| Limite | Solicitação com contato incompleto | Pendência em `aguardando dados`, retomável pelo identificador | Comercial completa com histórico ou cancela com motivo |
| Falha | Tentativa de encerrar sem responsável ou de gerar cotação | Portal nega encerramento ou ação proibida | Item permanece aberto; Comercial corrige atribuição |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** SPEC-F1-001, SPEC-F1-002, `02-Escopo-Definitivo.md` §§3 e 5/Fase 1 e esta SPEC.
2. **Alterar somente:** estados permitidos, painel de fila, campos de exceção e permissões da Comercial no ambiente de homologação.
3. **Não alterar:** catálogo ativo, regras de preço, alçadas financeiras, bots, notificações externas, integrações e transições de fases 2–5.
4. **Executar nesta ordem:** criar motivos/campos → restringir transições → configurar fila → aplicar permissões → testar item fora da grade, falta de dados e decisão terminal.
5. **Parar e pedir validação quando:** a configuração permitir cotação/alteração de catálogo a partir da exceção, uma notificação externa for requerida ou não houver dono para a fila.
6. **Estado válido ao parar:** itens já abertos mantêm motivo, contexto e responsável; nenhuma nova regra ou comunicação externa é disparada.

## Checklist de execução

- [ ] `aguardando dados` e `em exceção humana` têm motivo, contexto, responsável e próxima ação obrigatórios.
- [ ] Fila permite filtro por estado e responsável sem expor preço, proposta ou OP.
- [ ] Apenas Comercial classifica ou encerra; cliente/lead não muda estado operacional.
- [ ] Sem responsável, a exceção permanece aberta e visível.
- [ ] Decisão registrada preserva histórico; só recusa ou cancelamento é terminal e não apaga a solicitação.
- [ ] Cenários de item fora da grade, dado faltante e ação proibida têm evidência.

## Critérios de aceite

- [ ] **CA-1-10:** item fora da grade gera `em exceção humana` com motivo, contexto, responsável e próxima ação.
- [ ] **CA-1-11:** dado obrigatório ausente mantém `aguardando dados` e a solicitação pode ser retomada pelo identificador.
- [ ] **CA-1-12:** uma exceção não pode gerar cotação, preço, proposta, mudança de catálogo ou compromisso de produção.
- [ ] **CA-1-13:** decisão registrada preserva histórico, autor e timestamp; apenas recusa/cancelamento encerra a solicitação, e ausência de responsável impede esse encerramento.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Criar item fora da grade, campo vazio e tentativa de ação proibida | No ambiente de homologação, usar solicitação de teste e conta Comercial | Antes das restrições, motivo/responsável podem faltar ou ação indevida pode ser possível | Capturas do comportamento anterior |
| GREEN | Configurar campos obrigatórios, transições e fila | Repetir os três cenários e registrar orientação, recusa e cancelamento | Exceção/pendência ficam rastreáveis; orientação preserva `em exceção humana`; ação proibida é negada | Capturas da fila, histórico e mensagem de bloqueio |
| REFACTOR/REGRESSÃO | Retomar e testar permissões de cliente | Retomar um item, tentar alteração com conta Cliente e encerrar sem responsável | Histórico persiste; cliente é negado; encerramento é bloqueado | Roteiro datado e matriz de permissões |

**Dados/fixtures:** solicitação de teste de SPEC-F1-001, item fora da grade de SPEC-F1-002, campo de contato vazio e contas Cliente/Comercial.  
**Caminhos de erro obrigatórios:** ausência de dado, item fora da grade, responsável ausente, permissão negada e tentativa de cotação/alteração de catálogo.  
**Evidência exigida:** capturas da fila e dos históricos, além de roteiro assinado com os quatro resultados.

## Handoff e operação

- **Como demonstrar:** criar uma exceção, atribuir Comercial, registrar uma decisão e mostrar que não surgiu cotação ou alteração de grade.
- **Como operar depois:** Comercial revisa a fila e registra cada ação no portal; sinais recorrentes seguem para análise de catálogo sem mudança direta.
- **Como monitorar:** volume por motivo, itens sem responsável e idade das pendências; a definição de SLA formal ocorre em fase posterior.
- **Pendência conhecida:** alçadas, SLA, política de frete/amostra e retorno autorizado a cotação continuam fora da Fase 1.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T07 | Configurar fila, campos obrigatórios e transições de exceção | Administrador do portal | F1-003 | CA-1-10, CA-1-13 | Pendência por dado ausente, exceção e acesso Cliente | Capturas de fila, campos e acesso negado | F1-T01 concluída | Planejada — após F1-T01 |
| F1-T08 | Restringir compromisso automático e registrar decisões de exceção | Comercial | F1-003 | CA-1-10 a CA-1-13 | Orientação, recusa, cancelamento e tentativa proibida | Histórico, estados e bloqueio | F1-T05 e F1-T07 concluídas | Planejada — após F1-T05/F1-T07 |
| F1-T09 | Executar prova final de fila de exceção e handoff operacional | Comercial | F1-003 | CA-1-10 a CA-1-13 | RED/GREEN/REGRESSÃO e handoff da SPEC | Roteiro datado e confirmação operacional | F1-T08 concluída | Planejada — após F1-T08 |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
