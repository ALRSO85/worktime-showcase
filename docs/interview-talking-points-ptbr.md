# WorkTime — pontos de conversa para entrevista

Roteiro de entrevista baseado nas decisões documentadas neste showcase público e sanitizado.

## Resumo em 30 segundos

> O WorkTime consolida apontamentos internos e dados de integrações como Businessmap/Kanbanize e Ahgora em fluxos de jornada, conciliação, fechamento e auditoria. Sou responsável pelo desenho, pela arquitetura e pela construção do produto. Um desafio importante é lidar com escritas externas quando a resposta é perdida ou ambígua: o sistema precisa distinguir uma falha confirmada de um resultado desconhecido antes de decidir o que fazer.

## Case: escrita idempotente e confirmação incerta

### Cenário e papel

- O produto combina apontamentos internos e externos e precisa manter histórico e rastreabilidade.
- Sou responsável pelo desenho, pela arquitetura e pela construção do produto.

### Desafio e decisão

- **Desafio:** um provedor pode processar um apontamento e a resposta não chegar à aplicação. Tratar toda ausência de resposta como falha e reenviar às cegas pode duplicar a operação.
- **Decisão:** reservar uma chave idempotente estável antes da chamada externa e modelar separadamente falha confirmada e resultado incerto.

### Funcionamento

1. O WorkTime registra a operação como `Pending` antes do envio.
2. Envia a escrita ao provedor uma vez.
3. Em sucesso confirmado, persiste o estado, a tentativa, o log de sincronização e a auditoria.
4. Em falha confirmada, marca a tentativa como `Failed`.
5. Sem confirmação confiável, marca `Unknown`, não repete automaticamente a escrita e exige conferência no provedor antes de reconciliar ou tentar de novo.

Retries automáticos ficam limitados a falhas transitórias em `GET` e `HEAD`; a política compartilhada não repete operações de escrita.

### Consequência técnica e trade-off

O estado `Unknown` preserva a incerteza em vez de transformar uma chamada ambígua em falha segura para reenvio. Isso reduz o risco de duplicidade causada por retry cego, mas exige conferência e reconciliação antes de uma nova escrita. A decisão privilegia correção e rastreabilidade sobre repetição automática indiscriminada.

### Resposta curta sugerida

> No fluxo de integração, uma resposta ausente não prova que o provedor deixou de processar a solicitação. Por isso, reservamos a operação com uma chave idempotente antes do envio. Se o resultado for ambíguo, registramos `Unknown` e não repetimos a escrita automaticamente; primeiro é preciso conferir o estado no provedor. Mantemos retries transitórios para leituras `GET` e `HEAD`, cuja semântica é diferente. O ponto principal é modelar a incerteza explicitamente, sem afirmar entrega exatamente uma vez ou ganhos quantitativos que não medimos.

## Perguntas para aprofundar

1. O que diferencia `Failed` de `Unknown` no fluxo?
2. Por que a reserva acontece antes da chamada externa?
3. Como a operação é reconciliada após a conferência no provedor?
4. Quais leituras podem ser repetidas e por que essa política não se aplica às escritas?

## Limites do material público

O showcase não publica código completo, credenciais, dados de usuários, detalhes privados de provedores nem métricas de negócio. Não afirmar redução medida de duplicidades, economia de tempo ou disponibilidade sem fonte autorizada.
