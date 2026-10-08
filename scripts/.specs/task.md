# TAREFAS

Este arquivo condensa as tarefas executáveis baseadas no contexto e regras de negócio presentes nos arquivos `spec.md` e `plan.md`.

## TAREFA 1 - Constantes, Tempo e Cálculo de Valor
- [ ] Declarar as variáveis e constantes de negócio no código.
- [ ] Implementar a lógica da fração de cobrança: `VALOR_FRACAO_CENTAVOS = TARIFA_HORA_CENTAVOS * FRACAO_MINUTOS // 60`.
- [ ] Criar validação do campo `entrada`: rejeitar datas sem fuso horário ou fora do padrão ISO-8601 (retornar erro `entrada_invalida`).
- [ ] Criar validação de data para os relatórios: rejeitar valores fora do formato `AAAA-MM-DD` (retornar erro `data_invalida`).
- [ ] Implementar a lógica de cálculo de tempo: contabilizar apenas minutos completos.
- [ ] Implementar a regra de tolerância: se `minutos <= TOLERANCIA_MINUTOS`, o valor cobrado deve ser `0`.
- [ ] Garantir que o cálculo financeiro armazene e retorne o resultado sempre como número inteiro (formato centavos) no campo `valor_centavos`.

## TAREFA 2 - Armazenamento e Estados
- [ ] Implementar regra de unicidade de vaga: se uma placa já tem um bilhete em aberto, rejeitar a criação de um novo (retornar erro `bilhete_em_aberto`).
- [ ] Garantir que ao encerrar ou cancelar um bilhete, a placa seja imediatamente liberada para criar novos bilhetes.
- [ ] Impedir o duplo encerramento: tentar encerrar um bilhete já fechado deve retornar erro `bilhete_ja_encerrado`.
- [ ] Impedir o encerramento de um bilhete cancelado ou o cancelamento de um bilhete já encerrado (retornar erro `bilhete_nao_aberto`).
- [ ] Validar a existência do bilhete nas rotas paramétricas: se o ID não existir, retornar erro `bilhete_nao_encontrado`.
- [ ] Configurar o payload de um bilhete cancelado: garantir que ele não possua os campos `saida`, `minutos` nem `valor_centavos`.

## TAREFA 3 - Rotas HTTP (Built-in)
- [ ] Configurar o servidor HTTP para rodar no endereço `0.0.0.0:PORTA_SERVICO`.
- [ ] Garantir que **todas** as respostas do servidor (sucesso ou erro) sejam retornadas com `Content-Type: application/json`.
- [ ] Criar rota `POST /bilhetes`: retornar status `201` em caso de sucesso na abertura.
- [ ] Criar rota `POST /bilhetes/{id}/encerramento`: retornar status `200` junto aos campos `id`, `placa`, `entrada`, `saida`, `minutos` e `valor_centavos`.
- [ ] Criar rota `POST /bilhetes/{id}/cancelamento`: retornar status `200` com payload `{"status": "cancelado"}`.
- [ ] Criar rota `GET /bilhetes/ativos`: retornar status `200`. *(Atenção: o roteador deve capturar esta rota antes da rota paramétrica `/bilhetes/{id}`)*.
- [ ] Criar rota `GET /bilhetes?placa=`: retornar status `200` contendo a lista de bilhetes ou um array vazio `[]` caso a placa nunca tenha estacionado.
- [ ] Criar rota `GET /relatorios/diario?data=`: retornar status `200` com os indicadores financeiros e de volume do dia.
- [ ] Padronizar payloads de erro para o formato `{"erro": "<codigo_do_erro>"}`, utilizando rigorosamente as chaves definidas no `spec.md`.
- [ ] Implementar a precedência de erros: validações sintáticas (`422`, ex: placa inválida) devem ser processadas e retornadas antes de validações de regra de negócio (`409`, ex: conflito de bilhete).
- [ ] Configurar um manipulador (handler) global para rotas não mapeadas, retornando status `404` com o payload `{"erro": "rota_nao_encontrada"}`.
- [ ] **Garantir restrição tecnológica:** a aplicação deve rodar executando apenas `python app.py`, sem a necessidade de instalar nenhuma dependência externa (nenhum `pip install` necessário).