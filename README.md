
# DetSegPay API Documentation

## Base URL
Todas as requisições devem ser feitas para o seguinte URL base:
```
https://api.detsegpay.com/v1
```

## Autenticação
Para autenticar as requisições da API, obtenha um token de acesso utilizando `client_id`, `client_token` e `client_environment`. O token é utilizado no cabeçalho de autorização para todas as requisições subsequentes.

### Requisição para gerar o token

**Endpoint**: `POST /token`

**Cabeçalhos**:
```
Content-Type: application/json
```

**Corpo**:
```json
{
  "client_id": "e064bdcc-3f9f-40a4-9745-997b369b2",
  "client_token": "URDOFCD2J4FAUNR3UY7KWBPS52Y1TVXEVVIJKOX8I4VARMIFEPW1WNDB9LT4ZZZZI38D",
  "client_environment": "prd"
}
```

**Resposta**:
```json
{
  "access_token": "<JWT_ACCESS_TOKEN>",
  "expires_in": 1740951022
}
```

### Exemplo de cURL

```bash
curl -X POST https://api.detsegpay.com/v1/token -H "Content-Type: application/json" -d '{
  "client_id": "your_client_id",
  "client_token": "your_client_token",
  "client_environment": "prd"
}'
```

## Criar Transação
Crie uma transação utilizando o token de acesso autenticado.

### Requisição para criar uma transação

**Endpoint**: `POST /transactions/buy`

**Cabeçalhos**:
```
Content-Type: application/json
Authorization: Bearer <JWT_ACCESS_TOKEN>
```

**Corpo**:
```json
{
  "identification": "1234567890",
  "full_name": "NOME COMPLETO",
  "email": "usuario@email.com",
  "phone_number": "+5511999887766",
  "description": "QUALQUER INFO",
  "amount": 1
}
```

### Códigos de Resposta

- **201 Created**: PIX gerado com sucesso.
- **404 Not Found**: Token de acesso expirado.

## Consulta de Transações

**Endpoint**: `POST /transactions/query`

Necessário estar autenticado. Para consultar uma transação específica ou um período de transações.

### Exemplo de consulta de uma transação específica

**Corpo**:
```json
{
  "transaction_id": "221d0d07-8c98-4743-9f23-18af51f278c2"
}
```

**Resposta**:
```json
{
  "transactions": [
    {
      "transaction_id": "221d0d07-8c98-4743-9f23-18af51f278c2",
      "status": "released",
      "value": 1.0,
      "created_at": "2025-03-03T17:13:57.718059Z"
    }
  ]
}
```

### Exemplo de consulta de um período de transações

**Corpo**:
```json
{
  "start_date": "2025-01-01T00:00:00",
  "end_date": "2025-03-04T23:59:59"
}
```

**Resposta**:
```json
{
  "transactions": [
    {
      "transaction_id": "3e80dfe4-ecbb-42d2-a57c-b71fc8b265b4",
      "status": "pending",
      "value": 1000.0,
      "created_at": "2025-03-03T03:09:29.832665Z"
    },
    ...
  ]
}
```

## Tratamento de Erros

Em caso de erro, a API irá responder com o código HTTP apropriado e um objeto JSON contendo detalhes sobre o erro.

**Exemplo de resposta de erro**:
```json
{
  "error": "invalid_request",
  "message": "Mensagem de erro detalhada."
}
```

### Erros Comuns

- **400 Bad Request**: Parâmetros inválidos ou formato de requisição incorreto.
- **401 Unauthorized**: Token de acesso inválido ou expirado.
- **500 Internal Server Error**: Erro inesperado no servidor.
- **403 Forbidden: Acesso negado (ex.: IP não autorizado ou conta banida).
- **404 Not Found: Recurso não encontrado (rota incorreta ou usuário/carteira inexistente).
- 

## Rate Limiting

A API possui limites de taxa para evitar abuso. Ultrapassar o limite resultará em um erro **429 Too Many Requests**.
