# WorkTime — showcase técnico

> Visão pública e sanitizada do produto. O código-fonte, configurações internas e dados operacionais não fazem parte deste material.

## Visão geral

WorkTime é uma aplicação web para gestão e conciliação de horas. Reúne apontamentos internos, dados do Businessmap/Kanbanize e importações do Ahgora em fluxos de trabalho com jornadas, equipes, fechamentos, férias, relatórios e auditoria.

[Acessar a aplicação WorkTime](https://worktime.devforyou.com.br/account/login) — o acesso às áreas operacionais requer uma conta autorizada.

## Desafio de produto

Consolidar apontamentos que chegam de fontes diferentes e dar suporte às rotinas de equipes, jornadas, férias e fechamentos, preservando histórico e rastreabilidade.

## Minha contribuição

Sou responsável pelo desenho, pela arquitetura e pela construção do produto.

## Abordagem

O WorkTime reúne integrações e registros internos em fluxos de trabalho organizados por módulos. A separação entre apresentação, casos de uso, persistência e integrações permite evoluir cada área de forma incremental.

## Arquitetura em alto nível

- **Aplicação:** ASP.NET Core MVC/Razor sobre .NET 10.
- **Persistência:** Entity Framework Core e SQL Server.
- **Infra e publicação:** aplicação em container Docker no Ubuntu Server, com Cloudflare Tunnel no caminho de acesso.
- **Organização:** monólito modular, com fronteiras para áreas como identidade, apontamentos, integrações e notificações.
- **Integrações:** Businessmap/Kanbanize para cartões e sincronização de apontamentos; Ahgora para consulta semanal e importação de horas.

```mermaid
flowchart LR
    UI[Interface web] --> Modules[Módulos de negócio]
    Modules --> Integrations[Businessmap e Ahgora]
    Modules --> Data[EF Core e SQL Server]
```

## Engenharia

A aplicação evolui de forma incremental. Serviços de aplicação concentram casos de uso; testes automatizados e verificações arquiteturais ajudam a proteger contratos e fronteiras entre módulos.

### Integrações resilientes

Leituras externas usam retries seletivos para falhas transitórias. Operações que registram apontamentos não são repetidas automaticamente: cada envio reserva uma chave idempotente antes da chamada e resultados incertos exigem conferência no provedor antes de um novo envio. Isso mantém explícita a diferença entre falha confirmada e confirmação externa desconhecida.

```mermaid
sequenceDiagram
    participant App as WorkTime
    participant DB as SQL Server
    participant Provider as Provedor externo
    App->>DB: Reservar chave estável como Pending
    DB-->>App: Reserva confirmada
    App->>Provider: Enviar apontamento uma vez
    alt Sucesso confirmado
        Provider-->>App: Resultado confirmado
        App->>DB: Persistir estado, tentativa, log de sincronização e auditoria
    else Falha confirmada
        Provider-->>App: Falha confirmada
        App->>DB: Marcar tentativa como Failed
    else Resultado incerto
        Provider--x App: Resposta ausente ou ambígua
        App->>DB: Marcar tentativa como Unknown
        Note over App,Provider: Não reenviar automaticamente. Verificar o provedor e reconciliar antes de novo envio
    end
```

Retries automáticos se limitam a `GET` e `HEAD` transitórios; requisições de escrita ficam fora da política compartilhada.

## Segurança e auditoria

O produto inclui autenticação baseada em cookies, proteção antiforgery, limitação de requisições e registros de auditoria. Este resumo não publica detalhes de configuração, rotas administrativas ou informações de infraestrutura.

## Escopo deste showcase

Para preparar uma conversa técnica em português, consulte os [pontos de conversa para entrevista](docs/interview-talking-points-ptbr.md).

Este documento descreve capacidades e decisões em nível de produto. Não inclui código-fonte, credenciais, dados de usuários, regras comerciais internas, topologia detalhada ou configurações operacionais privadas.
