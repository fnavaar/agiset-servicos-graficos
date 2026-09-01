# Controle de aprendizado contínuo

> Registro interno das triagens; append-only, em ordem cronológica inversa.
> Formato: `- <ISO-8601> · task <ID|nenhuma> · <status> · <motivo>`

- 2026-09-01T21:22:00Z · task F1-T02 · capturado:06_notas/aprendizado-continuo/AP-2026-09-01-2120-corpo-hook-409.md · corpo de hook PocketBase chega em err.response do SDK JS, não em err.response.data; toast genérico mascarou 409 correto (evidência: teste browser antes/depois)
- 2026-09-01T18:38:35Z · task F1-T01 · capturado:06_notas/aprendizado-continuo/AP-2026-09-01-1535-imutabilidade-hook.md · autogeneratePattern não garante imutabilidade no PocketBase; exige request hook (evidência: PATCH não alterou valor)
- 2026-09-01T18:28:00Z · task F1-T01 · sem sinal reutilizável · falha inicial de cadastro = fluxo de auth ausente, já documentado na correção (não é novo padrão)