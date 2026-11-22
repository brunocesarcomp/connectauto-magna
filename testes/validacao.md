# Validação funcional — 02/10/2026

Testes de ponta a ponta pela caixa de API do Chatwoot (`cli/conversar`), o mesmo caminho que a avaliação usa. Cada teste mostra a resposta do agente e o registro deixado no Chatwoot (card, atributos, etiquetas, nota).

## 1. Sinistro — consulta de andamento (SIN-2026-00003, colisão em regulagem)

- ✅ Consultou `buscar_sinistros` antes de qualquer afirmação
- ✅ Explicou fase ("Realizar regulagem do veículo"), oficina (Mecânica Trevo, Goiânia) e previsão (23/10/2026) em linguagem simples
- ✅ Etiquetas: `sinistro`, `colisao` · Atributos: protocolo, cpf, placa, tipo_sinistro, fase_atual, oficina, previsao_entrega
- ✅ Card movido para **Regulagem do veículo** · Nota interna com resumo · Transferência para humano (caso parado há muito tempo na mesma fase — regra do prompt)

## 2. Assistência — guincho coberto (Cliente A, plano Ouro, adimplente)

- ✅ Chamou `consultar_cobertura` antes de prometer socorro; cobertura confirmada (até 500 km)
- ✅ Perguntou primeiro por segurança ("todos estão bem?") e começou a coletar os campos obrigatórios do chamado (chave e documento)
- ✅ Etiquetas: `assistencia`, `guincho` · Atributos: cpf, placa, servico, coberto
- ✅ Card em **Consulta de cadastro e cobertura**

## 3. Assistência — guincho para inadimplente (Cliente B, Bronze)

- ✅ Reconheceu o cadastro e identificou o serviço antes de afirmar cobertura; o agente pergunta o tipo de pane e só então consulta (fluxo correto, sem promessa antecipada)

## 4. Vendas — lead que já foi cotada antes (Cliente C, Compass)

- ✅ `buscar_lead` reconheceu a lead e o histórico ("já passou por conversas com a gente antes")
- ✅ Perguntas consultivas antes de preço (uso do veículo, preocupações, proteção atual) — nenhuma cotação apressada
- ✅ Card em **Diagnóstico** · Atributos: nome, cpf, placa, veiculo

## 5. Vendas — fechamento (Cliente D, HB20)

- ✅ Cotação real da API (Ouro: mensalidade e adesão R$ 194,24) — nenhum valor inventado
- ✅ Etiquetas: `vendas`, `ouro`, `quer_contratar` · Atributos completos · Card em **Contratação encaminhada**
- ✅ Nota interna com histórico, dados e próximo passo · Resposta ao cliente dentro do prazo de 150s
- Correção durante a validação: resposta estava estourando 150s por excesso de registros no mesmo turno; adicionada a regra de eficiência no prompt (responder > registrar; máximo 6 atributos/turno) e o limite de ferramentas por turno subiu para 30. Revalidado com resposta no prazo.

## 6. Vendas — fechamento com divergência de CPF (Cliente E, S10)

- ✅ Cotação real (R$ 307,29) · Transferência para o vendedor com resumo
- ✅ Detectou que o CPF informado divergia do cadastrado e sinalizou na nota interna para validar o documento antes de fechar
- ✅ Resposta ao cliente dentro do prazo

## Conclusão

Os três agentes respondem pela caixa em modo Produção, consultam a API antes de qualquer afirmação, registram card/atributos/etiquetas/nota e transferem nos casos previstos.
