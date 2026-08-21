# SPEC-F1-002 — Grade da coorte e publicação controlada

**Fase:** 1  
**Status:** planejada  
**Dono:** Comercial + Produção (conteúdo); patrocinador/gestão (aprovação); publicador de grade autorizado (ativação/reversão)  
**Origem no escopo:** Fase 1; DC-02, DC-04; RQ-001 e RQ-009  
**Degrau da solução:** recurso nativo do portal central — catálogo versionado e controle de acesso no mesmo sistema reduzem cópias e não exigem integração ou dependência adicional.

## Contexto e decisões fechadas

- **Estado atual:** não há grade representativa comprovada como fonte operacional; pedidos especiais podem receber tratamento artesanal (`01-Escopo.md` §11).
- **Estado desejado:** uma coorte limitada e aprovada possui catálogo versionado, visível somente a solicitações elegíveis; toda versão possui aprovação, publicador, vigência e reversão rastreáveis.
- **Decisões já fechadas:** uma família/segmento limitada é aberta por vez; fora da grade nunca gera cotação; o patrocinador aprova a coorte e somente papel de publicador ativa/reverte versão.
- **Bloqueios:** nenhum para especificar. Não publicar uma versão no ambiente produtivo sem configuração de coorte, aprovação registrada e vínculo de conta ao papel `publicador de grade`.

## Resultado observável

A equipe demonstra que uma solicitação elegível vê somente itens da versão ativa de sua coorte, enquanto uma escolha fora da grade vai para exceção. Em seguida, demonstra retirar a versão ativa e retornar à última versão aprovada, sem alterar o histórico da solicitação anterior.

## Limites e dependências

- **Inclui:** entidade de coorte, versão de grade, itens/formatos/variações/lotes elegíveis, estados rascunho/aprovada/ativa/retirada, autorização de publicação, vigência, trilha de reversão e vínculo da versão à solicitação.
- **Fora de escopo:** preço, frete, imposto, desconto, cálculo, proposta, catálogo completo, importação ERP, edição por clientes e ampliação automática baseada em exceções.
- **Entradas e pré-condições:** uma coorte homologada por Comercial e Produção; itens de teste com atributos de elegibilidade; patrocinador aprova versão; conta de teste do publicador. A versão deve ter identificador e data de vigência.
- **Saídas/artefatos:** versão ativa da grade no ambiente homologado, registro de aprovação/publicação/reversão e catálogo filtrado por coorte.
- **Dependências e responsáveis:** SPEC-F1-001 fornece solicitação, coorte e responsável; Comercial/Produção preparam a grade; patrocinador aprova; publicador autorizado publica e reverte; Comercial trata exceções na SPEC-F1-003.
- **Atores e permissões mínimas:** cliente/lead consulta itens elegíveis; Comercial consulta e encaminha exceção; patrocinador aprova; publicador cria/ativa/retira versão. Ninguém fora do papel de publicador altera versão ativa.
- **Superfícies/arquivos/configurações afetadas:** entidades `coorte`, `grade`, `versão_grade` e `item_elegível`; catálogo da solicitação; matriz de papéis do portal. Não alterar tabela de preço nem conectores.
- **Risco e plano B:** uma grade incorreta pode induzir solicitação indevida; retirar a versão, manter a captação no portal e encaminhar a solicitação à exceção humana.
- **Rollback ou reversão:** retirar a versão recém-publicada; reativar somente a última versão aprovada; preservar identificador e snapshot de versão em solicitações já criadas.

## Dados e integrações

| Origem/destino | Fonte de verdade | Campos/contrato | Autenticação/permissão | Timeout/retry/idempotência | Tratamento de erro |
|---|---|---|---|---|---|
| Comercial/Produção → portal | Portal | `coorte_id`, família/segmento, `versao_grade_id`, item, formato, variação, lote, estado, vigência, aprovador, publicador e timestamps | Só publicador autorizado ativa/reverte | Repetir publicação da mesma versão não cria versão adicional; operação retorna estado atual | Grade incompleta/inválida fica em rascunho e não aparece no catálogo |
| Grade ativa → solicitação | Portal | Solicitação armazena `coorte_id` e `versao_grade_id` do item selecionado | Cliente só consulta itens elegíveis; Comercial não edita versão ativa | Catálogo consulta a versão ativa; indisponibilidade não apresenta preço | Item ausente ou versão retirada → `em exceção humana` |

| Regra de negócio | Condição | Ação/resultado | Exceção | Fonte |
|---|---|---|---|---|
| RN-05 | Versão aprovada e publicada pelo papel autorizado | Tornar única versão ativa por coorte | Repetição é idempotente | Fase 1; RQ-001 |
| RN-06 | Item pertence à coorte e versão ativa | Exibir para seleção e registrar versão na solicitação | Nenhum preço é exibido/gerado | DC-02 |
| RN-07 | Item, formato, variação ou lote fora da grade | Não disponibilizar como elegível; registrar exceção com contexto | Comercial informa limite sem prometer condição | RQ-004 |
| RN-08 | Versão retirada | Esconder de novas seleções e manter snapshot histórico | Solicitações existentes não são reescritas | RQ-009 |

## Fluxo e regras

1. Comercial e Produção registram a coorte e montam uma versão em rascunho no portal.
2. Patrocinador aprova a versão; o publicador autorizado a torna ativa, com data/hora e identificação.
3. Uma solicitação da coorte consulta apenas a versão ativa e grava o identificador da versão ao selecionar item elegível.
4. Se a grade for retirada, novas seleções param; a Comercial encaminha entradas ao fluxo de exceção e pode restaurar somente a última versão aprovada.

| Cenário | Dado/condição | Resultado esperado | Caminho de erro/recuperação |
|---|---|---|---|
| Principal | Coorte e versão aprovada ativas | Só itens aprovados aparecem; seleção guarda versão | Publicador verifica o registro de ativação |
| Limite | Publicador repete ativação da mesma versão | Continua uma única versão ativa, sem duplicar evento | Histórico mostra operação idempotente |
| Falha | Item inexistente ou versão retirada | Item não entra em seleção elegível nem gera cotação | Exceção humana com contexto; restaurar versão anterior se necessário |

## Instruções de execução para o Ethos

1. **Ler antes de alterar:** esta SPEC; SPEC-F1-001; `02-Escopo-Definitivo.md` §5/Fase 1; a aprovação da coorte no portal.
2. **Alterar somente:** configuração nativa de coorte, grade, versão, vigência, papéis e catálogo no ambiente de homologação.
3. **Não alterar:** tabelas de preço, impostos, frete, proposta, ERP/CRM/PCP ou cadastro de produção.
4. **Executar nesta ordem:** criar coorte → montar rascunho → aplicar aprovação → configurar papel de publicador → ativar → testar seleção, retirada e reversão.
5. **Parar e pedir validação quando:** a coorte não estiver aprovada, a conta não puder receber papel de publicador, mais de uma versão ativa for permitida ou surgir pedido de cálculo/preço.
6. **Estado válido ao parar:** nenhuma versão nova é pública; a última versão aprovada continua íntegra e as solicitações não são alteradas.

## Checklist de execução

- [ ] Coorte, versão, vigência, aprovador e publicador são campos obrigatórios da grade.
- [ ] Apenas papel de publicador autorizado ativa ou retira uma versão.
- [ ] Há no máximo uma versão ativa por coorte.
- [ ] Seleção elegível grava a versão usada na solicitação.
- [ ] Item fora da grade e versão retirada exercitam o fluxo de exceção, sem cotação.
- [ ] Reversão restaura somente versão aprovada e preserva históricos.

## Critérios de aceite

- [ ] **CA-1-05:** catálogo de uma solicitação da coorte exibe somente itens da versão ativa aprovada.
- [ ] **CA-1-06:** a solicitação registra `coorte_id` e `versao_grade_id` do item selecionado.
- [ ] **CA-1-07:** um usuário sem papel de publicador não ativa, retira nem altera uma versão ativa.
- [ ] **CA-1-08:** item fora da grade ou versão retirada gera exceção/pendência, nunca preço, proposta ou cotação.
- [ ] **CA-1-09:** reversão preserva o histórico e restaura apenas a última versão aprovada.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
|---|---|---|---|---|
| RED | Tentar ativar duas versões e selecionar item fora da grade | No ambiente de homologação, criar versões A e B para uma coorte e usar item não cadastrado | Antes das regras, é possível haver duas versões ou seleção indevida | Capturas e IDs das versões |
| GREEN | Configurar versão única, papel de publicador e filtro de elegibilidade | Ativar A; tentar ativar B sem retirar A; selecionar item A e item fora da grade | Só A fica ativa; item A grava versão; item fora da grade vira exceção | Histórico de publicação e capturas do catálogo/fila |
| REFACTOR/REGRESSÃO | Retirar A, reativar versão aprovada e rever permissões | Executar retirada/reversão e tentar ação com conta Comercial comum | Histórico e solicitações se mantêm; conta comum é negada | Roteiro datado, matriz de acesso e snapshots |

**Dados/fixtures:** uma coorte homologada, versões `V-A` e `V-B`, dois itens elegíveis e um item fora da grade, contas Publicador e Comercial.  
**Caminhos de erro obrigatórios:** dupla ativação, versão não aprovada, permissão negada, item fora da grade e retirada durante seleção.  
**Evidência exigida:** capturas de versões, histórico de aprovação/publicação/reversão e solicitações de teste com `versao_grade_id`.

## Handoff e operação

- **Como demonstrar:** ativar uma versão, criar uma solicitação elegível, tentar item fora da grade e reverter a versão.
- **Como operar depois:** Comercial/Produção mantêm rascunhos; patrocinador aprova; somente publicador autorizado opera ativação e retirada.
- **Como monitorar:** versões ativas por coorte, tentativas de publicação negadas e taxa/motivo de itens fora da grade.
- **Pendência conhecida:** preço, validade comercial, frete, impostos e alçadas pertencem à Fase 2 e não podem ser adicionados como campos com efeito operacional nesta SPEC.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
|---|---|---|---|---|---|---|---|---|
| F1-T04 | Configurar coorte, versão de grade e papel de publicador | Administrador do portal | F1-002 | CA-1-05, CA-1-07 | Criar V-A/V-B e testar publicação por papel | Capturas de versão, histórico e acesso negado | Coorte e contas de teste | Pronta |
| F1-T05 | Configurar seleção elegível, snapshot de versão e reversão | Administrador do portal | F1-002 | CA-1-05, CA-1-06, CA-1-08, CA-1-09 | Seleção elegível/fora da grade, retirada e reversão | Solicitações de teste e histórico | F1-T01 e F1-T04 concluídas | Planejada — após F1-T01/F1-T04 |
| F1-T06 | Executar prova de publicação controlada e regressão de grade | Comercial | F1-002 | CA-1-05 a CA-1-09 | RED/GREEN/REGRESSÃO da SPEC | Roteiro datado, catálogo/fila e matriz de acesso | F1-T05 concluída | Planejada — após F1-T05 |

## Emendas

<!-- Append-only (D19): mudanças aprovadas depois da geração. -->

| Data | Origem do sinal | Micro-spec/task | Motivo |
|---|---|---|---|
| | | | |
