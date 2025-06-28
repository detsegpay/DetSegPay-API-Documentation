Integração com a API DetsegPay 

(Python)

A DetsegPay é uma plataforma de pagamento que oferece soluções completas via API para:
- Geração de cobranças via Pix;
- Consulta de transações por ID ou período;
- Verificação do status de pagamentos.
Este guia fornece uma integração genérica com a API DetsegPay utilizando Python 3.7+, com foco em robustez,
clareza e boas práticas.

2. Pré-Requisitos
   
- Conta ativa na DetsegPay.
- Credenciais válidas (client_id, client_token).
- Liberação de IP (se necessário).
- Ambiente com Python 3.7 ou superior.

- 3. Instalação do Ambiente
     
python -m venv venv
source venv/bin/activate
venv\Scripts\activate
# Linux/macOS
# Windows
pip install aiohttp asyncio requests

4. Base URL da API
BASE_URL = "https://api.detsegpay.com/v1"

5. Métodos Disponíveis
   
- authenticate(): Realiza a autenticação e armazena o token.
- create_payment(): Cria um pagamento Pix com os dados do cliente.
- verify(): Verifica o status do pagamento criado.
- query_transaction(): Consulta transações por ID ou intervalo de datas.

- 6. Exemplo de Uso
 
detseg = DetsegPay(client_id="SEU_CLIENT_ID", client_token="SEU_CLIENT_TOKEN", id="USUARIO_ID")
token = await detseg.authenticate()
qr_code = await detseg.create_payment(
identification="CPF OU CNPJ",
full_name="João da Silva",
email="joao@email.com",
phone_number="5511987654321",
description="Assinatura Premium",
amount=10000
)
status = await detseg.verify()
transactions = await detseg.query_transaction(transaction_id=detseg.id_pagamento)

Exemplo Curl
curl --location 'https://api.detsegpay.com/v1/transactions/buy' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzgyMzM0NTAyLCJpYXQiOjE3NTA3OTg1MDIsImp0aSI6IjZlZmUwMzJkYjhjMzRjMDJhMjJhZmRhZDIyNGJjNzhlIiwiY2xpZW50X2lkIjoiMzk3NWYxZjUtMWQ2MS00NmIyLWI3ZGItZDk3N2QxMjU2MDNiIn0.zaQsR_2okYhQqJQxStFEMS2xKDbMiLRGjvl-aCAM3Yw' \
--header 'Content-Type: application/json' \
--data-raw '{
    "identification": "42402381884",
    "full_name": "Marlon de Oliveira Poloni Chagas",
    "email": "marlon.poloni@consultoriadevcore.com.br",
    "phone_number": "11971408114",
    "amount": 1
}'

Exemplo Python

import requests
import json

url = "https://api.detsegpay.com/v1/transactions/buy"

payload = json.dumps({
  "identification": "42402381884",
  "full_name": "Marlon de Oliveira Poloni Chagas",
  "email": "marlon.poloni@consultoriadevcore.com.br",
  "phone_number": "11971408114",
  "amount": 1
})
headers = {
  'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzgyMzM0NTAyLCJpYXQiOjE3NTA3OTg1MDIsImp0aSI6IjZlZmUwMzJkYjhjMzRjMDJhMjJhZmRhZDIyNGJjNzhlIiwiY2xpZW50X2lkIjoiMzk3NWYxZjUtMWQ2MS00NmIyLWI3ZGItZDk3N2QxMjU2MDNiIn0.zaQsR_2okYhQqJQxStFEMS2xKDbMiLRGjvl-aCAM3Yw',
  'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)

7. Lógica de Retentativas
   
import asyncio
async def criar_pagamento_seguro(pays, value):
for tentativa in range(10):
try:
cpf, nome, email, telefone = next(datauser())
qr = await pays.create_payment(cpf, nome, email, telefone, "Pix DetsegPay", value)
if qr:
return qr
except Exception as e:
print(f"Erro tentativa {tentativa+1}: {e}")
await asyncio.sleep(3)
return None

8. Boas Práticas
   
- Valide credenciais antes de iniciar o fluxo.
- Trate exceções com try-except.
- Revalide token automaticamente se expirar.
- Use logs estruturados para facilitar a depuração.
- Use retry com backoff exponencial (opcional).
- Certifique-se de que seu IP está liberado na API.

  Código Completo da Classe
  
import aiohttp
BASE_URL = "https://api.detsegpay.com/v1"
class DetsegPay:
def __init__(self, client_id, client_token, id):
self.client_id = client_id
self.client_token = client_token
self.id = id
self.token = None

self.id_pagamento = None
async def authenticate(self):
async with aiohttp.ClientSession() as session:
url = f"{BASE_URL}/auth"
async with session.post(url, json={
"client_id": self.client_id,
"client_token": self.client_token
}) as resp:
data = await resp.json()
self.token = data.get("access_token")
return self.token
async def create_payment(self, identification, full_name, email, phone_number, description,
amount):
async with aiohttp.ClientSession() as session:
url = f"{BASE_URL}/payment/pix"
headers = {"Authorization": f"Bearer {self.token}"}
payload = {
"client_id": self.client_id,
"id": self.id,
"identification": identification,
"full_name": full_name,
"email": email,
"phone_number": phone_number,
"description": description,
"amount": amount
}
async with session.post(url, headers=headers, json=payload) as resp:
data = await resp.json()
self.id_pagamento = data.get("id")
return data.get("qrcode")
async def verify(self):
async with aiohttp.ClientSession() as session:
url = f"{BASE_URL}/payment/status"
headers = {"Authorization": f"Bearer {self.token}"}
payload = {"id": self.id_pagamento}
async with session.post(url, headers=headers, json=payload) as resp:
return await resp.json()
async def query_transaction(self, transaction_id=None, start_date=None, end_date=None):
async with aiohttp.ClientSession() as session:
url = f"{BASE_URL}/payment/search"
headers = {"Authorization": f"Bearer {self.token}"}
payload = {"client_id": self.client_id}
if transaction_id:
payload["id"] = transaction_id
elif start_date and end_date:
payload["start_date"] = start_date

payload["end_date"] = end_date
async with session.post(url, headers=headers, json=payload) as resp:
return await resp.json()

Geração do Pix

{
    "message": "Código PIX gerado com sucesso",
    "transaction_id": "962356c0-69e6-493c-b155-6c3e8e59c284",
    "payment_code": "00020126820014br.gov.bcb.pix2560pix.s3bank.com/qr/v3/at/1f2692ec-9e51-42b7-80c4-05c7105c73a65204000053039865802BR5925DETSEGINFOBR_SERVICOS_DIG6009SAO_PAULO62070503***63040687",
    "payment_qr_code": "iVBORw0KGgoAAAANSUhEUgAAAjoAAAI6AQAAAAAGM99tAAAFYUlEQVR4nO2dQW7jSAxFP6cMZCkBc4AcpXSDOVKQm0lHyQEaKC0DWOAsqsiinF517Onq8a+FEdvygwQwxNcvkhLFXdb21304AEEEEUQQQQQRRBBBBP1fQNLWBQCO+je2GcA2H+2LbQbCX8tuP1oecUYEEfTNpaqqyKqqWpIC0xW6IikwqaoWOwRTPw5ALknDb9fxLo0gggDsLfmqlqTyVg5RLYDIq6osfvg2HwJM1/rOsvxDzogggu4JEpkBbHNSbPMhquUQfZ8P0RWH6GqH6PpfnRFBBP3Kuty8V+wCBQ7R/PGi2BZAgBcV4ALJBSpA+uqCj3dpBBGEqqlXmPyQOSmAQwAkE9aqqu9yAfJH+39Q1evDzogggu4A2kSkCpH80WJXFiSVtwLIAgDYL5C30n4ni3sojzkjggj63tLbVZrUUC1Jda05O2nPz9FN8UVvhKCxQGbpAUAu/uIqIxcAQKpapd435u4OTlcwsgkaERRydg1WLQCqulZtOXudLJnnkrTFc0nR42ZkEzQWyCK72NuehtsmTYv2GNlAvaFE+A8Y79IIempQjOJ1UlcZGgK4SexJTYCr53bmbIIGBZlMthtF5NJjt91BIuu1Sux281gA3NiBjGyCxgKFO0jL2f4CoAV6cTWiGnQ27yAJGhbkObsFcM/ZzQJB0iBJun1S66WUaoSgMUEW2e7hrbXcqfl65vC5SwJYmR+9EYJGBp12atz4qBHbY7dEPV6/cG+EkU3QiKCgRlqwmqndXOyqN07Z++ymUGcTNDAoF6CWO2G/tFJW4BBZcIi+y0sTJ9gvrc9m8ZvMx5wRQQR9bzVvxByRViNSzPrr249nP7st5myCBgXd6mzzrvsmOsyxntoLYGK795IxsgkaDBQ2X0I9SNboWIc99SpJulVCP5ugQUFek+oCo9sdiE1hlq7hRjfaPSe9EYIGBHnOPhl58F3GILuLhbfHM/1sgoYHufGB6VOQS1KpXTNz66lpXey7SDtuj31jI18aQU8JssjcZ9i0haTY5h8XBRTYFkCxpyu25XJFXgFgRzt4e2UfJEFjgryKNYUSqDhYp650KpWyeSOhZYxqhKCxQF43crL+fFCUbU7aHmSol6o/87/GuzSCnhoUcvZpw7zXZ5/KAVvdyKmEmzmboAFBX3J23JABAG9JiPWsbUXAeJdG0FODTrvrwKl3ffJiKDV1nb0ZUm2vkmqEoBFBIWfbBqOHMnCayjBpECf0swn6c0D546WK7To623tqVMshdWD2gqRANbVFLO4fdkYEEfTL60Ywd1/PikKuCB2+LXH3jkjmbILGBMV5I9mb1YOBjT74rN9LAugDWRnZBA0MqhL7s82f7B3r63SFLPaZyOunhDK/4KGMe2kEPSXodD8Ia2DHTeyaLokjSXq9FHM2QeOBzjr76tV8550aj2cfSdIUCuuzCRoUFHdq1vPYp/7wpRro9TOfPEw/m6CRQadJOuqV2sXnZ/ctHDsujNgJgPEujaCnBrkasXTdR40Es9p2asJT8wCw1o+gcUG3zzwA+ojKoFCi/OiChd1iBA0Lis88AHzT0XTJT82Q1dM6Z0QRNCro9hm+Vuvnm459fyb4f6wbIWh0UByJHQb5nVy/etx5/Ej90BuDGdkEDQb6SWSf9hab4rYGyaxeDliAXhjIyCZoMNDtM3yRVyiwCwCkq2D/WyUXKLZ/CnSTpMB+Qe3/BY67nxFBBN0TNJlZXatTq944xGpJDpvFYM29suwXYJMX1o0QNCToizeSbdOxrcntwMltvoKT2KYaIWg8UFUjfV6wtpderHrzBVyIxJ+NeGkEEUQQQQQRRBBBBBH020D/AsvbniZHUkDwAAAAAElFTkSuQmCC"
}
payment_qr_code -> qr_code pro PIX
payment_code -> código copia e cola



Integração com a API DetsegPay (Node.js)


1. Introdução

Este documento fornece um guia completo para integrar a API da DetsegPay utilizando Node.js. O foco está em
robustez, tratamento de erros, boas práticas de autenticação e exemplos de código claros para facilitar a
implementação.


2. Pré-Requisitos
Antes de iniciar, certifique-se de ter os seguintes requisitos:
- Conta ativa na plataforma DetsegPay.
- Credenciais válidas:
- DETSEGPAY_CLIENT_ID
- DETSEGPAY_CLIENT_TOKEN
- Node.js instalado em sua máquina.
- Pacotes NPM necessários: axios, dotenv

  3. Instalação dos Pacotes
     
npm init -y
npm install axios dotenv
Crie um arquivo .env para armazenar suas credenciais de forma segura:
DETSEGPAY_CLIENT_ID=seu_client_id
DETSEGPAY_CLIENT_TOKEN=seu_client_token

4. Base URL da API
const BASE_URL = "https://api.detsegpay.com/v1";

5. Classe DetsegPay (Node.js)
   
Crie um arquivo chamado detsegpay.js com o seguinte conteúdo:

const axios = require('axios');
require('dotenv').config();
class DetsegPay {
constructor(userId) {
this.BASE_URL = "https://api.detsegpay.com/v1";
this.clientId = process.env.DETSEGPAY_CLIENT_ID;
this.clientToken = process.env.DETSEGPAY_CLIENT_TOKEN;
this.token = null;
this.transactionId = null;
this.paymentCode = null;
this.userId = userId;
}

async authenticate() {
const url = `${this.BASE_URL}/token`;
try {
const response = await axios.post(url, {
client_id: this.clientId,
client_token: this.clientToken,
client_environment: "prd"
});
if (response.status === 200 && response.data.access_token) {
this.token = response.data.access_token;
return this.token;
}
throw new Error('Token inválido');
} catch (error) {
console.error("Erro na autenticação:", error.message);
return null;
}
}
async createPayment({ identification, fullName, email, phoneNumber, description, amount }) {
if (!this.token && !(await this.authenticate())) {
return null;
}
const url = `${this.BASE_URL}/transactions/buy`;
const headers = {
Authorization: `Bearer ${this.token}`,
"Content-Type": "application/json"
};
const payload = {
identification,
full_name: fullName,
email,
phone_number: `+${phoneNumber}`,
description,
amount
};
try {
const response = await axios.post(url, payload, { headers });
if (response.status === 201 && response.data.transaction_id) {
this.transactionId = response.data.transaction_id;
this.paymentCode = response.data.payment_code;
return this.paymentCode;
}
return null;

} catch (error) {
console.error("Erro ao criar pagamento:", error.message);
return null;
}
}
async verifyStatus() {
if (!this.token && !(await this.authenticate())) return null;
if (!this.transactionId) return null;
const url = `${this.BASE_URL}/transactions/query`;
const headers = {
Authorization: `Bearer ${this.token}`,
"Content-Type": "application/json"
};
try {
const response = await axios.post(url, {
transaction_id: this.transactionId
}, { headers });
const transactions = response.data.transactions || [];
if (transactions.length === 0) return "ERRO: Nenhuma transação encontrada.";
const status = transactions[0].status;
switch (status) {
case 'pending': return 'PENDENTE';
case 'paid': return 'PAGO';
case 'liquidated': return 'PAGO (aguardando PIX-OUT)';
case 'rejected': return 'RECUSADO';
default: return 'ERRO';
}
} catch (error) {
console.error("Erro ao verificar status:", error.message);
return 'ERRO';
}
}
async queryTransaction({ transactionId, startDate, endDate }) {
if (!this.token && !(await this.authenticate())) return null;
const url = `${this.BASE_URL}/transactions/query`;
const headers = {
Authorization: `Bearer ${this.token}`,
"Content-Type": "application/json"
};
const payload = transactionId ? { transaction_id: transactionId } :
(startDate && endDate) ? { start_date: startDate, end_date: endDate } :
null;
if (!payload) return "ERRO: Parâmetros insuficientes para consulta.";
try {
const response = await axios.post(url, payload, { headers });
return response.data.transactions || [];
} catch (error) {
console.error("Erro ao consultar transações:", error.message);
return 'ERRO';
}
}
}
module.exports = DetsegPay;

6. Exemplo de Uso com Retentativas
Crie um arquivo index.js com o exemplo de uso:
const DetsegPay = require('./detsegpay');
(async () => {
const detseg = new DetsegPay("123456789");
const maxRetries = 10;
let paymentCode = null;
for (let attempt = 1; attempt <= maxRetries; attempt++) {
paymentCode = await detseg.createPayment({
identification: "CPF OU CNPJ",
fullName: "João da Silva",
email: "joao@email.com",
phoneNumber: "11999999999",
description: "Pagamento Teste",
amount: 10000
});
if (paymentCode) break;
console.log(`Tentativa ${attempt} falhou. Retentando em 3 segundos...`);
await new Promise(r => setTimeout(r, 3000));
}
if (paymentCode) {
console.log("PIX gerado com sucesso:");
console.log(paymentCode);
} else {
console.log("Falha Falha ao gerar pagamento após várias tentativas.");
}
})();


7. Boas Práticas

   
- Valide as credenciais antes de iniciar o fluxo.
- Trate exceções com blocos try/catch.
- Revalide o token automaticamente se ele expirar.
- Use logs estruturados para facilitar o rastreamento de problemas.
- Implemente retentativas com backoff exponencial, se necessário.
- Certifique-se de que seu IP está autorizado na API DetsegPay (se aplicável).

  Integração com a API DetsegPay (PHP)
  
1. Introdução
Este documento fornece um guia completo para integrar a API da DetsegPay utilizando
PHP. O foco está em robustez, tratamento de erros, boas práticas de autenticação e
exemplos de código claros para facilitar a implementação.

2. Pré-Requisitos
   
Antes de iniciar, certifique-se de ter os seguintes requisitos:
●​ Conta ativa na plataforma DetsegPay.​
●​ Credenciais válidas:​
○​ DETSEGPAY_CLIENT_ID​
○​ DETSEGPAY_CLIENT_TOKEN​
●​ PHP 7.4 ou superior instalado.​
●​ Composer instalado para gerenciamento de dependências.​
●​ Biblioteca Guzzle (cliente HTTP).​

3. Instalação das Dependências
   
Execute os comandos abaixo no terminal para inicializar o projeto e instalar as
dependências:
composer init
composer require guzzlehttp/guzzle vlucas/phpdotenv
Crie um arquivo .env para armazenar suas credenciais de forma segura:
DETSEGPAY_CLIENT_ID=seu_client_id
DETSEGPAY_CLIENT_TOKEN=seu_client_token

4. Base URL da API:
   
$baseUrl = "https://api.detsegpay.com/v1";

6. Classe DetsegPay (PHP)
Crie um arquivo chamado DetsegPay.php com o
seguinte conteúdo:
<?php
require 'vendor/autoload.php';
use GuzzleHttp\Client;
use Dotenv\Dotenv;
class DetsegPay {
private $baseUrl;
private $clientId;
private $clientToken;
private $token;
private $transactionId;
private $paymentCode;
private $client;
public function __construct() {
$dotenv = Dotenv::createImmutable(__DIR__);
$dotenv->load();
$this->baseUrl = "https://api.detsegpay.com/v1";
$this->clientId = $_ENV['DETSEGPAY_CLIENT_ID'];
$this->clientToken = $_ENV['DETSEGPAY_CLIENT_TOKEN'];
$this->client = new Client();
}
public function authenticate() {
$url = "{$this->baseUrl}/token";
try {
$response = $this->client->post($url, [
'json' => [
'client_id' => $this->clientId,
'client_token' => $this->clientToken,
client_environment' => 'prd'
]
]);
$data = json_decode($response->getBody(), true);
$this->token = $data['access_token'] ?? null;
return $this->token;
} catch (Exception $e) {
echo "Erro na autenticação: " . $e->getMessage();
return null;
}
}
public function createPayment($data) {
if (!$this->token && !$this->authenticate()) return null;
$url = "{$this->baseUrl}/transactions/buy";
try {
$response = $this->client->post($url, [
'headers' => [
'Authorization' => "Bearer {$this->token}",
'Content-Type' => 'application/json'
],
'json' => $data
]);
$body = json_decode($response->getBody(), true);
$this->transactionId = $body['transaction_id'] ?? null;
$this->paymentCode = $body['payment_code'] ?? null;
return $this->paymentCode;
} catch (Exception $e) {
echo "Erro ao criar pagamento: " . $e->getMessage();
return null;
}
}
}
?>
6. Exemplo de Uso
Crie um arquivo index.php com o seguinte conteúdo:
<?php
require_once 'DetsegPay.php';
$detseg = new DetsegPay();
$paymentCode = $detseg->createPayment([
'identification' => 'CPF OU CNPJ',
'full_name' => 'João da Silva',
'email' => 'joao@email.com',
'phone_number' => '+5511999999999',
'description' => 'Pagamento Teste',
'amount' => 10000
]);
if ($paymentCode) {
echo "PIX gerado com sucesso: $paymentCode";
} else {
echo "Falha ao gerar pagamento.";
}
?>

7. Boas Práticas
●​ Valide as credenciais antes de iniciar o fluxo.​
●​ Trate exceções com blocos try/catch.​
●​ Revalide o token automaticamente se ele expirar.​
●​ Use logs estruturados para facilitar o rastreamento de problemas.​
●​ Implemente retentativas com backoff exponencial, se necessário.​
●​ Certifique-se de que seu IP está autorizado na API DetsegPay (se aplicável).
