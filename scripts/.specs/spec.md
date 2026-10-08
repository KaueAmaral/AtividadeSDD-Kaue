# Especificação do Projeto (Spec)
**Projeto:** API de Estacionamento Rotativo
**Descrição:** API RESTful para gestão de estacionamento rotativo. O sistema permite abrir e encerrar bilhetes por placa, listar bilhetes ativos, gerar histórico por placa e emitir relatórios diários de faturamento e uso. O sistema opera exclusivamente via API (sem back-office).

---

## 1. Regras e Variáveis de Negócio Gerais
O cálculo de valores e o comportamento do sistema dependem das seguintes constantes (que deverão ser definidas nas configurações da aplicação):

*   `FRACAO_MINUTOS`: Unidade de tempo para cobrança fracionada.
*   `TARIFA_HORA_CENTAVOS`: Valor cobrado por uma hora cheia (em centavos). O valor da fração é calculado por: `Tarifa Hora / (60 / FRACAO_MINUTOS)`.
*   `TETO_DIARIO_CENTAVOS`: Valor máximo a ser cobrado por um bilhete em um único dia.
*   `TOLERANCIA_MINUTOS`: Tempo de isenção. Se o tempo de permanência for menor ou igual à tolerância, o valor é zero. Caso ultrapasse, a cobrança é integral (a tolerância não é descontada do tempo cobrado).
*   **Formato Financeiro:** O sistema nunca utiliza ponto flutuante. Todos os valores monetários trafegam e são armazenados em números inteiros (centavos).

---

## 2. Casos de Uso e Endpoints

### UC01 - Abrir Bilhete
**User Story:** Como operadora de estacionamento, preciso fazer a abertura de bilhetes para registrar a entrada de veículos.

*   **Endpoint:** `POST /bilhetes`
*   **Request Body:**
    ```json
    {
      "placa": "ABC1D23",
      "entrada": "2026-10-07T10:00:00-03:00" 
    }
    ```
    *A propriedade `entrada` é opcional e serve para testabilidade. Se omitida, usa-se o "agora". A placa deve conter 7 caracteres alfanuméricos maiúsculos.*
*   **Response (201 Created):**
    ```json
    {
      "id": 1,
      "placa": "ABC1D23",
      "entrada": "2026-10-07T10:00:00-03:00",
      "status": "aberto"
    }
    ```
*   **Regras Específicas (UC08 - Vaga Única):** Não é permitido abrir bilhete se já existir um bilhete com `status="aberto"` para a mesma placa. (Retorna `409 Conflict`).

### UC02 - Encerrar Bilhete
**User Story:** Como operadora de estacionamento, preciso encerrar bilhetes ativos para liberar a placa e calcular o valor devido.

*   **Endpoint:** `POST /bilhetes/{id}/encerramento`
*   **Response (200 OK):**
    ```json
    {
      "id": 1,
      "placa": "ABC1D23",
      "entrada": "2026-10-07T10:00:00-03:00",
      "saida": "2026-10-07T11:35:00-03:00",
      "minutos": 95,
      "valor_centavos": 1250
    }
    ```
*   **Regras Específicas:**
    *   Cobra-se por fração (arredondando para cima). Uma fração exata cobra 1 fração; 1 minuto a mais inicia a cobrança da próxima fração.
    *   Aplica-se a regra de `TOLERANCIA_MINUTOS` (UC07).
    *   Aplica-se o teto máximo `TETO_DIARIO_CENTAVOS`.

### UC03 - Listar Bilhetes Ativos
**User Story:** Como operadora, preciso listar todos os bilhetes que estão atualmente ocupando vagas (abertos e não encerrados).

*   **Endpoint:** `GET /bilhetes/ativos`
*   **Response (200 OK):** Array de objetos de bilhetes abertos, ordenados do mais recente para o mais antigo.

### UC04 - Relatório Diário
**User Story:** Como operadora, preciso visualizar o faturamento total, tempo médio e contagem de bilhetes encerrados em uma data específica.

*   **Endpoint:** `GET /relatorios/diario?data=AAAA-MM-DD`
*   **Response (200 OK):**
    ```json
    {
      "data": "2026-10-05",
      "total_bilhetes": 12,
      "faturamento_centavos": 8400,
      "tempo_medio_minutos": 47
    }
    ```
*   **Regras Específicas:** O `tempo_medio_minutos` considera apenas bilhetes encerrados no dia consultado. O valor deve ser arredondado (0,5 ou superior arredonda para cima).

### UC05 - Cancelar Bilhete
**User Story:** Como operadora, preciso cancelar um bilhete em aberto em caso de erro operacional, sem gerar cobranças.

*   **Endpoint:** `POST /bilhetes/{id}/cancelamento`
*   **Response (200 OK):**
    ```json
    {
      "id": 1,
      "status": "cancelado"
    }
    ```
*   **Regras Específicas:** Somente bilhetes em status `aberto` podem ser cancelados. O cancelamento não gera data de saída e nem valor cobrado.

### UC06 - Histórico por Placa
**User Story:** Como operadora, preciso consultar o histórico de todas as passagens de um veículo específico pelo estacionamento.

*   **Endpoint:** `GET /bilhetes?placa={placa}`
*   **Response (200 OK):** Array de todos os bilhetes (qualquer status) da placa informada, ordenados do mais recente para o mais antigo. Se a placa nunca estacionou, retorna array vazio `[]`.

---

## 3. Tratamento de Erros (Status Codes)

Todos os erros de validação ou regra de negócio devem retornar um JSON no formato `{"erro": "codigo_do_erro"}` conforme mapeamento abaixo:

| Situação | HTTP Status | Body (JSON) |
| :--- | :--- | :--- |
| Placa ausente, formato inválido | `422 Unprocessable Entity` | `{"erro": "placa_invalida"}` |
| Entrada fora do formato ISO-8601 | `422 Unprocessable Entity` | `{"erro": "entrada_invalida"}` |
| Parâmetro de data fora de AAAA-MM-DD | `422 Unprocessable Entity` | `{"erro": "data_invalida"}` |
| ID de bilhete não encontrado no BD | `404 Not Found` | `{"erro": "bilhete_nao_encontrado"}` |
| Tentativa de encerrar bilhete não aberto | `409 Conflict` | `{"erro": "bilhete_ja_encerrado"}` |
| Tentativa de cancelar bilhete não aberto | `409 Conflict` | `{"erro": "bilhete_nao_aberto"}` |
| Abrir bilhete com placa já ativa | `409 Conflict` | `{"erro": "bilhete_em_aberto"}` |