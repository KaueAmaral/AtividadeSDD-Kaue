# Plano de Implementação (Plan)

**Projeto:** API de Estacionamento Rotativo
**Stack Tecnológica:** Python (Exclusivamente Biblioteca Padrão - `http.server`, `sqlite3` ou memória, `datetime`, `json`)

## 1. Estratégia de Implementação e Arquitetura

Como o projeto não utilizará frameworks externos (como FastAPI, Flask ou Django), a arquitetura exigirá a criação de um roteador manual simples estendendo `BaseHTTPRequestHandler`. 

As fases lógicas de implementação sugeridas são:
*   **Fase 1:** Setup do servidor HTTP base e estruturação do armazenamento (Banco de dados em memória ou SQLite).
*   **Fase 2:** Implementação das regras de negócio centrais e cálculo de tempo/valor.
*   **Fase 3:** Roteamento e criação dos endpoints de Abertura, Encerramento e Cancelamento.
*   **Fase 4:** Criação dos endpoints de Listagem e Relatório.

## 2. Modelagem de Dados

O sistema possui uma única entidade principal que gerencia todo o ciclo de vida do estacionamento.

**Entidade:** `BILHETE`

| Atributo | Tipo (Python) | Descrição / Regras |
| :--- | :--- | :--- |
| `id` | `int` | Identificador único autoincremental. |
| `placa` | `str` | Placa do veículo (7 caracteres, alfanumérico, UPPERCASE). |
| `entrada` | `datetime` | Data e hora de abertura do bilhete (formato ISO-8601). |
| `saida` | `datetime` | Data e hora de encerramento do bilhete. Nulo enquanto aberto/cancelado. |
| `status` | `enum` (`str`) | Estado atual do bilhete. Valores aceitos: `"aberto"`, `"encerrado"`, `"cancelado"`. |
| `minutos` | `int` | Tempo total de permanência. Nulo enquanto aberto/cancelado. |
| `valor_centavos` | `int` | Valor final a ser pago. Nulo enquanto aberto/cancelado. |

## 3. Mapeamento de Endpoints da API

Abaixo está o mapeamento técnico das rotas que deverão ser interceptadas e tratadas pelo nosso Handler HTTP.

### Fluxo de Operação
*   **`POST /bilhetes`**
    *   **Ação:** Cria um novo bilhete.
    *   **Sucesso:** `201 Created`
    *   **Restrição:** Retorna `409 Conflict` se a placa enviada já possuir um bilhete em `status="aberto"`.
*   **`POST /bilhetes/{id}/encerramento`**
    *   **Ação:** Finaliza um bilhete aberto, registrando saída, calculando minutos e gerando o valor (centavos).
    *   **Sucesso:** `200 OK`
*   **`POST /bilhetes/{id}/cancelamento`**
    *   **Ação:** Anula um bilhete aberto sem gerar cobrança.
    *   **Sucesso:** `200 OK` (retornando `{"status": "cancelado"}`).

### Fluxo de Consulta e Relatórios
*   **`GET /bilhetes/ativos`**
    *   **Ação:** Retorna todos os bilhetes com `status="aberto"`.
    *   **Sucesso:** `200 OK` (array ordenado pelos mais recentes primeiro).
*   **`GET /bilhetes?placa={placa}`**
    *   **Ação:** Busca histórico completo de uma placa específica.
    *   **Sucesso:** `200 OK` (array ordenado pelos mais recentes primeiro).
*   **`GET /relatorios/diario?data=AAAA-MM-DD`**
    *   **Ação:** Consolida dados dos bilhetes encerrados na data informada.
    *   **Sucesso:** `200 OK` (contém total de bilhetes, faturamento e tempo médio).