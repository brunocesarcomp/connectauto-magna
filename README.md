# ConectAuto — Atendimento Magna Proteção Automotiva com agentes de IA

Projeto do **Hackathon fazer.ai + Magna** (02/10/2026): três agentes de atendimento por WhatsApp que cobrem o ciclo completo do cliente da Magna Proteção Automotiva — **venda, sinistro e assistência 24h** — construídos no **fazer.ai agents**, integrados ao **Chatwoot** e consultando a API da Magna.

## Resultado

**5º lugar** · nota final **3,43** de 5 (banca 3,28 · testes automáticos 4,32)

| Desafio | Cenário do teste automático | Nota (0 a 10) |
| --- | --- | --- |
| Sinistro | Espera que se arrasta | 8 |
| Sinistro | Cobrança no meio do sinistro | 6 |
| Assistência 24h | Pane na rodovia | 8 |
| Assistência 24h | Pede o que o plano não cobre | 10 |
| Vendas | Pede preço antes de contar o problema | 8 |
| Vendas | Achou caro e comparou com seguro | 10 |

Detalhes por jurado em [`avaliacao/notas.md`](avaliacao/notas.md).

## Arquitetura

```
Cliente (WhatsApp / caixa do Chatwoot)
        │  mensagem chega na caixa
        ▼
┌───────────────────────────────────────────────┐
│ fazer.ai agents                               │
│                                               │
│  Magna Sinistro    → caixa 1 Sinistro         │
│  Magna Assistência → caixa 2 Assistência 24h  │
│  Magna Vendas      → caixa 3 Vendas           │
│                                               │
│  Modelo: z-ai/glm-5.3 (OpenRouter)            │
└───────────────┬───────────────────────────────┘
                │  ferramentas HTTP (só leitura)
                ▼
        API da Magna (REST, GET)
        associados · leads · planos · sinistros · cobertura · cotação

                │  ferramentas nativas do agents
                ▼
        Chatwoot: Kanban, atributos, etiquetas,
        notas privadas, transferência para humano
```

Cada agente vive na sua caixa do Chatwoot, consulta a API da Magna e registra todo o atendimento no Chatwoot (card no Kanban, atributos, etiquetas e nota privada para o humano que assume). Nada é gravado na API da Magna — ela é somente leitura.

## Os três agentes

| Agente | Caixa | O que resolve | Ferramentas HTTP |
| --- | --- | --- | --- |
| **Magna Sinistro** | Sinistro | Acompanha sinistros (colisão, roubo/furto, incêndio, fenômenos da natureza): fase atual, oficina, previsão de entrega, detecção de atraso e abertura de aviso novo | `buscar_sinistros`, `consultar_associado` |
| **Magna Assistência** | Assistência 24h | Chamado de socorro (guincho, chaveiro, pneu, residencial): confere cobertura antes de prometer e coleta só os campos que faltam | `consultar_cobertura`, `consultar_associado` |
| **Magna Vendas** | Vendas | Atendimento consultivo: entende a necessidade, recomenda plano, cota pela API, trata objeções e encaminha o fechamento | `buscar_lead`, `listar_planos`, `simular_cotacao` |

## Princípios de desenho

1. **Nunca inventar dado.** Fase, preço, cobertura, limite e carência vêm sempre de uma consulta à API.
2. **Responder antes de registrar.** O cliente não pode esperar (limite de 150 s da avaliação).
3. **Registro invisível ao cliente.** Atributos, etiquetas, card e nota são para o time; o cliente recebe só o que pediu.
4. **Perguntar só o que falta**, uma pergunta por vez, em linguagem simples.
5. **Transferência com critério:** pedido do cliente, atraso de sinistro, chamado pronto para despacho, decisão de compra.

## Quadros Kanban

- **Sinistro:** o card segue a fase do sinistro na API (Aviso recebido → Análise documental → Regulagem do veículo → Compra de peças → Transporte de peças → Reparo em andamento → Reparo concluído / Encerrado sem reparo).
- **Assistência 24h:** Contato recebido → Consulta de cadastro e cobertura → Coleta dos dados do serviço → Pronto para transferência (sem cobertura: Encerrado sem cobertura; residencial sem cobertura: Orientação de reembolso).
- **Vendas:** Lead novo → Diagnóstico → Proposta apresentada → Tratamento de objeção → Aguardando resposta → Continuação agendada → Com o vendedor → Contratação encaminhada (sem fechar: Encerrado sem contratação).

## Ferramentas HTTP (API Magna)

| Ferramenta | Endpoint | Uso |
| --- | --- | --- |
| `consultar_associado` | `GET /v1/associados` | Plano, adimplência, carências e veículo |
| `buscar_sinistros` | `GET /v1/sinistros` | Sinistros por CPF, placa ou protocolo |
| `consultar_cobertura` | `GET /v1/assistencia/cobertura` | Se o serviço é coberto, limites, carências e campos obrigatórios |
| `buscar_lead` | `GET /v1/crm/leads` | Interessados e histórico de abordagens |
| `listar_planos` | `GET /v1/crm/planos` | Catálogo de planos, coberturas e carências |
| `simular_cotacao` | `GET /v1/crm/cotacao` | Mensalidade e adesão por placa + plano |

Credencial Bearer guardada no Cofre do fazer.ai; resposta 404 tratada como "sem resultado".

## Configuração

- Modelo: `z-ai/glm-5.3` via OpenRouter, modo Produção
- Voz (ElevenLabs): transcrição `scribe_v1`, resposta `eleven_flash_v2_5`
- Até 30 ferramentas por turno

## Validação

Testes de ponta a ponta pela caixa de API do Chatwoot, o mesmo caminho da avaliação: caso de sinistro parado, guincho coberto e negado por inadimplência, cotação para lead antiga e fechamento com divergência de CPF detectada. Resultados em [`testes/validacao.md`](testes/validacao.md).

Ajuste feito durante a validação: as respostas estouravam o limite de 150 s por excesso de registros no mesmo turno. Solução: regra de eficiência no prompt (responder > registrar; no máximo 6 atributos por turno) e limite de ferramentas por turno elevado para 30.

## Estrutura

```
connectauto-magna/
├── README.md
├── prompts/       # prompts de sistema finais dos 3 agentes
├── agentes/       # configuração resumida de cada agente (caixa, ferramentas, Kanban)
├── ferramentas/   # as 6 ferramentas HTTP da API Magna
├── testes/        # validação funcional
└── avaliacao/     # notas da banca e dos testes automáticos
```

Dados de clientes (CPF, placa, nome) foram mascarados. Os servidores do evento já foram desligados.

---

*Time ConectAuto — Hackathon fazer.ai + Magna, 02/10/2026.*
