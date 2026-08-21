# SPEC-F1-001 — Registro único e retomada segura de solicitação

**Fase:** 1  
**Status:** planejada  
**Dono:** Comercial (operação); patrocinador/gestão (aprovação de acesso)  
**Origem no escopo:** Fase 1; DC-04; RQ-001, RQ-003 e RQ-009  
**Degrau da solução:** construção mínima no portal central aprovado — o escopo já fixa o portal como fonte de verdade; não há integração externa autorizada para a entrada inicial.

## Contexto e decisões fechadas

- **Estado atual:** solicitações chegam por canais dispersos, inclusive WhatsApp; o escopo define que o portal, e não a conversa, será a fonte de verdade (`02-Escopo-Definitivo.md` §§3 e 5).
- **Estado desejado:** cada lead ou atendimento iniciado pela Comercial possui uma solicitação única, recuperável e auditável no portal, sem cotação nesta fase.
- **Decisões já fechadas:** o portal é a fonte de verdade; WhatsApp só encaminha ao portal; estados permitidos nesta fase são `rascunho`, `aguardando dados` e `em exceção humana`; conflito de vínculo nunca é mesclado automaticamente.
- **Bloqueios:** nenhum para a SPEC. A autorização humana de 21/08/2026 confirma os gates operacionais da Fase 1. A implementação para imediatamente se não existir ambiente de homologação do portal ou se a permissão mínima não puder ser concedida.

## Resultado observável

A equipe demonstra criar uma solicitação da coorte, receber um identificador imutável, completar ou corrigir dados e retomá-la pela fila sem criar empresa, contato ou solicitação duplicados. O cliente/lead não vê preço, proposta ou compromisso de produção.

## Limites e dependências

- **Inclui:** registro de empresa, contato, origem, qualificação, seleção da coorte, estado, responsável, histórico de alteração e recuperação pela Comercial.
- **Fora de escopo:** cotação, tabela de preço, proposta, pagamento, arte, OP, PCP, sincronização com ERP/CRM e importação automática de contatos.
- **Entradas e pré-condições:** ambiente de homologação do portal; configuração de coorte aprovada; conta de teste para Comercial; política de acesso mínimo aplicada. Dados recebidos por WhatsApp devem ser digitados ou encaminhados ao portal, sem cópia automática.
- **Saídas/artefatos:** registro de solicitação, identificador, trilha de histórico, fila de retomada e evidência dos roteiros de teste.
- **Dependências e responsáveis:** patrocinador aprova a coorte e o acesso; Comercial é dona da qualificação e da retomada; quem administra o portal concede o papel de Comercial, sem conceder publicação de grade.
- **Atores e permissões mínimas:** cliente/lead cria ou completa somente seu registro; Comercial cria, pesquisa, retoma e classifica; patrocinador consulta e aprova a coorte. Nenhum desses papéis altera regras de catálogo por esta SPEC.
- **Superfícies/arquivos/configurações afetadas:** ambiente homologado do portal; entidades `empresa`, `contato`, `solicitação` e `histórico`; tela/formulário de entrada e fila da Comercial. Não alterar sistemas externos.
- **Risco e plano B:** indisponibilidade do portal ou dado incompleto gera `aguardando dados`; a Comercial registra o atendimento no próprio portal assim que ele retornar. Não manter uma planilha paralela como fonte de verdade.
- **Rollback ou reversão:** desativar a entrada pública, preservar registros e permitir somente criação/retomada assistida pela Comercial; não apagar histórico nem reutilizar identificadores.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Formulário/atendimento → portal | Portal | `solicitacao_id` gerado uma vez; empresa, contato, origem, segmento, modelo de negócio, coorte, estado, responsável, timestamps e histórico | Cliente limita-se ao próprio envio; Comercial usa papel operacional | Reenvio com mesma chave de reconhecimento deve localizar rascunho compatível; conflito não mescla | Dado obrigatório ausente → `aguardando dados`; conflito empresa/contato → revisão humana |
| WhatsApp → portal | Portal | Mensagem contém somente identificador/link de retomada ou registro manual pela Comercial | Nenhum acesso do WhatsApp ao banco do portal | Não há conector, webhook ou retry automático autorizado | Falha de encaminhamento → Comercial pesquisa/cria registro assistido e anota a origem |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-01 | Nova solicitação | Gerar `solicitacao_id` e iniciar `rascunho` | Falha de persistência não exibe confirmação de criação | RQ-003 |
| RN-02 | Campo obrigatório ausente | Manter `aguardando dados`; não liberar catálogo elegível | Comercial pode completar em nome do lead, com histórico | RQ-001 |
| RN-03 | Chave de contato/empresa encontra um rascunho compatível | Oferecer retomada pela Comercial | Mais de um candidato ou divergência abre revisão humana | RQ-003, RQ-009 |
| RN-04 | Item/dado fora de regra, identificado na triagem | Registrar `em exceção humana`, motivo e dona/o | Não produzir preço ou promessa | RQ-004 |

## Fluxo e regras

1. Cliente/lead ou Comercial abre uma solicitação e informa origem, empresa, contato, segmento e modelo de negócio.
2. O portal gera o identificador e valida campos mínimos; ausência mantém `aguardando dados`.
3. A Comercial consulta por identificador e chave de reconhecimento; se houver candidato único, retoma; se houver conflito, cria revisão humana registrada.
4. A Comercial atribui a coorte aprovada e o responsável atual. A seleção segue para o catálogo da SPEC-F1-002; toda não elegibilidade segue para a SPEC-F1-003.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Empresa e contato válidos, origem registrada | Solicitação única em `rascunho`, com responsável e histórico | Comercial conclui campos pendentes |
| Limite | Lead volta pelo WhatsApp com identificador | Comercial retoma a mesma solicitação no portal | Se o identificador não existir, registra novo atendimento com a origem preservada |
| Falha | Mesmo contato aponta para empresas divergentes | Nenhuma mesclagem ou cotação; revisão humana registrada | Comercial corrige vínculo e preserva o histórico |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** `02-Escopo-Definitivo.md` §§3 e 5; esta SPEC; a configuração de coorte aprovada no portal.
2. **Alterar somente:** entidades, campos, estados e telas listados nesta SPEC, no ambiente de homologação do portal.
3. **Não alterar:** preço, proposta, pagamento, arte, OP, PCP, ERP/CRM, conectores, políticas de retenção ou permissões além das mínimas.
4. **Executar nesta ordem:** criar entidades/campos → configurar estados e histórico → aplicar permissões → criar formulário/fila → exercitar RED/GREEN/regressão.
5. **Parar e pedir validação quando:** não houver ambiente homologado, não houver conta de teste, a coorte não estiver disponível para seleção ou uma integração externa for solicitada.
6. **Estado válido ao parar:** nenhum canal externo integrado; registros já criados continuam pesquisáveis e não recebem cotação.

## Checklist de execução

- [ ] Campos mínimos e `solicitacao_id` imutável existem no portal.
- [ ] Estados `rascunho`, `aguardando dados` e `em exceção humana` têm transições restritas.
- [ ] Comercial consegue pesquisar, retomar e registrar divergência sem mesclar dados.
- [ ] Cliente/lead não obtém permissão de editar regra ou histórico operacional.
- [ ] Roteiros principal, de reentrada e de conflito geraram evidência.

## Critérios de aceite

- [ ] **CA-1-01:** cada solicitação demonstrada contém identificador, empresa/contato, origem, estado, responsável e histórico.
- [ ] **CA-1-02:** reentrada por identificador retoma o registro e não cria uma segunda solicitação.
- [ ] **CA-1-03:** conflito de contato/empresa não é mesclado automaticamente e registra revisão humana.
- [ ] **CA-1-04:** nenhuma tela ou integração desta SPEC oferece cotação, preço ou envio à produção.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Reenviar a mesma entrada e simular conflito de empresa | No ambiente de homologação, criar `Empresa A/Contato X`, reenviar e depois tentar `Empresa B/Contato X` | Antes da configuração, há duplicidade ou vínculo sem revisão | Captura da falha e IDs criados |
| GREEN | Configurar chave de reconhecimento, retomada e revisão | Repetir o roteiro com massa `A/X`, `A/X` e `B/X` | Segundo envio retoma; terceiro abre revisão; não há cotação | Capturas da fila, histórico e estados |
| REFACTOR/REGRESSÃO | Confirmar campos ausentes e acesso mínimo | Criar solicitação sem contato e testar contas Cliente/Comercial | Registro fica em `aguardando dados`; só Comercial retoma/classifica | Roteiro assinado e matriz de permissões |

**Dados/fixtures:** `Empresa A`, `Contato X`, origem `WhatsApp`, coorte homologada, uma conta Cliente e uma conta Comercial de teste.  
**Caminhos de erro obrigatórios:** persistência falha, campo obrigatório vazio, reentrada, duplicidade e acesso não autorizado.  
**Evidência exigida:** capturas do formulário, fila, histórico e roteiro de teste datado no diretório de evidências da Fase 1.

## Handoff e operação

- **Como demonstrar:** criar uma solicitação, abandoná-la, retomá-la pelo identificador e registrar uma divergência de empresa/contato.
- **Como operar depois:** Comercial revisa a fila `aguardando dados` e as divergências a cada turno definido pela coorte.
- **Como monitorar:** quantidade de solicitações sem responsável, duplicidades evitadas e pendências sem resolução.
- **Pendência conhecida:** a política de retenção e incidente de dados segue como gate de ativação externa de D-09; esta SPEC não autoriza coleta pública antes de estar configurada.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T01 | Configurar registro mínimo, identificador imutável e acesso de homologação | Administrador do portal | F1-001 | CA-1-01, CA-1-04 | Criação `Empresa A/Contato X`; ID e acesso Cliente/Comercial | Capturas de formulário, permissões e registro | Ambiente e contas de teste | Pronta |
| F1-T02 | Configurar retomada por chave e revisão humana de conflito | Administrador do portal | F1-001 | CA-1-02, CA-1-03 | Roteiro `A/X`, `A/X`, `B/X` | Histórico e captura da revisão | F1-T01 concluída | Planejada — após F1-T01 |
| F1-T03 | Executar prova de regressão do registro e da retomada | Comercial | F1-001 | CA-1-01 a CA-1-04 | RED/GREEN/REGRESSÃO da SPEC | Roteiro datado e matriz de permissões | F1-T02 concluída | Planejada — após F1-T02 |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
