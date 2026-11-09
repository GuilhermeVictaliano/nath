# Arquitetura: Nath

## Visão geral

Toda mensagem recebida no WhatsApp dispara um cenário no Make, que decide o tipo de pedido e o encaminha. Um segundo cenário, agendado, verifica os lembretes que venceram e os envia.

```mermaid
flowchart TB
    subgraph Atendimento
        W[API do WhatsApp] -->|webhook: nova mensagem| M[Cenário principal]
        M --> R{Tipo de pedido}
        R -->|anotação| N[Grava anotação]
        R -->|lembrete| L[Grava lembrete: data, hora, destinatário]
        R -->|pergunta| AI[LLM]
        N --> DB[(Google Sheets / Drive)]
        L --> DB
        AI --> RESP[Resposta ao usuário]
        N --> RESP
        L --> RESP
        RESP --> W
    end
    subgraph Lembretes
        S[Agendamento] --> Q[Busca lembretes vencidos]
        DB --> Q
        Q --> E[Envia ao usuário ou ao terceiro]
        E --> W
    end
```

## O que é regra e o que é IA

| Etapa | Tipo |
|---|---|
| Receber a mensagem e responder | Regra (webhook + API do WhatsApp) |
| Gravar anotações e lembretes | Regra (Google Sheets / Drive) |
| Disparar lembretes no horário | Regra (agendamento) |
| Responder dúvidas | IA (OpenAI) |

## Decisões técnicas

- **WhatsApp como interface:** nada para instalar; o usuário já está lá.
- **Google Sheets como banco:** fácil de inspecionar e corrigir à mão, sem infraestrutura.
- **Make (no-code):** permitiu lançar rápido e iterar nos fluxos sem servidor próprio.
