# ConnectAuto — Atendimento Magna Proteção Automotiva

Agentes de atendimento por WhatsApp para a Magna, construídos no fazer.ai agents e integrados ao Chatwoot, no Hackathon fazer.ai + Magna (out/2026).

## Agentes
| Agente | Caixa | O que faz | Ferramentas da API |
|---|---|---|---|
| Magna Sinistro | 1 | Acompanha o sinistro, informa fase, oficina e previsão, percebe mudança de fase e transfere casos de atraso ou reclamação | buscar_sinistros, consultar_associado |
| Magna Assistência | 2 | Entende o pedido como o cliente escreve, confere cobertura, pergunta só os campos que faltam e deixa o chamado pronto para despacho | consultar_cobertura, consultar_associado |
| Magna Vendas | 3 | Conversa como consultor: entende a necessidade, recomenda plano, cota e registra objeções e decisão | buscar_lead, listar_planos, simular_cotacao |

## Princípios
- Consultar a API antes de afirmar qualquer dado; nunca inventar valor, fase ou cobertura.
- Registrar todo atendimento no Chatwoot: atributos na conversa, etiquetas, card no Kanban e nota privada para o humano.
- Transferir para humano quando o caso exige; encerrar a conversa quando termina.

## Configuração
- Modelo: z-ai/glm-5.3 (OpenRouter)
- Voz: ElevenLabs — transcrição scribe_v1, resposta eleven_flash_v2_5
- Ferramentas HTTP: GET na API da Magna, credencial Token Bearer no Cofre, 404 tratado como "sem resultado"
- Máximo de 30 ferramentas por turno

## Equipe
ConnectAuto
