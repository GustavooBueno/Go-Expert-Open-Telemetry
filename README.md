# Projeto clima por CEP (OTEL e Zipkin)

Este projeto é composto por dois serviços (A e B) que trabalham em conjunto para fornecer informações de temperatura com base em um CEP, além de integrar OpenTelemetry (OTEL) e Zipkin para observabilidade e tracing distribuído.

## Arquitetura do Sistema

### Serviço A
- Endpoint: `POST /input`
- Função: Recebe um CEP via requisição POST (JSON), verifica se possui um formato válido (8 dígitos) e, caso positivo, encaminha a solicitação para o Serviço B.
- Validações implementadas:
  - Verifica se o payload é um JSON válido
  - Valida se o CEP possui exatamente 8 dígitos numéricos

### Serviço B
- Endpoint: `GET /weather?cep={cep}`
- Função: Processa o CEP recebido, obtém a cidade correspondente através do ViaCEP, recupera a temperatura atual via WeatherAPI, converte os valores para Fahrenheit e Kelvin e retorna o resultado junto com o nome da cidade.
- Integrações:
  - ViaCEP (https://viacep.com.br/): Converte CEP em nome da cidade
  - WeatherAPI (https://www.weatherapi.com/): Fornece a temperatura atual da cidade

### Sistema de Observabilidade
Ambos os serviços geram spans de tracing e enviam para um Otel Collector, que exporta os dados para o Zipkin, possibilitando a análise detalhada dos traces e spans de cada requisição.

## Requisitos

- Docker
- Docker Compose
- Chave de API da [WeatherAPI](https://www.weatherapi.com/)

## Iniciar o projeto

1. Clone o repositório e entre no diretório do projeto:
   ```bash
   git clone https://github.com/GustavooBueno/Go-Expert-Open-Telemetry.git
   cd Go-Expert-Open-Telemetry
   ```

2. Obtenha uma chave de API gratuita da WeatherAPI:
   - Acesse [WeatherAPI](https://www.weatherapi.com/) e crie uma conta gratuita
   - Após o registro, obtenha sua chave API no dashboard

3. Ajuste as duas variáveis `WEATHER_API_KEY` no arquivo `docker-compose.yml` com a sua chave da WeatherAPI:
   ```yaml
   environment:
     - WEATHER_API_KEY=SUA_CHAVE_API_AQUI
   ```

4. Inicie a aplicação e as ferramentas de observabilidade:
   ```bash
   docker-compose up --build
   ```

## Testando a aplicação

### Exemplo de requisição básica

```bash
curl -X POST \
    -H "Content-Type: application/json" \
    -d '{"cep":"29102571"}' \
    http://localhost:8081/input
```

### Resposta esperada (exemplo)
```json
{
  "city": "Vila Velha",
  "temp_C": 25.0,
  "temp_F": 77.0,
  "temp_K": 298.0
}
```

### Outros exemplos de teste

1. CEP inválido (menos de 8 dígitos):
   ```bash
   curl -X POST \
       -H "Content-Type: application/json" \
       -d '{"cep":"123456"}' \
       http://localhost:8081/input
   ```
   Resposta esperada: `invalid zipcode` (Status 422)

2. CEP inválido (formato não numérico):
   ```bash
   curl -X POST \
       -H "Content-Type: application/json" \
       -d '{"cep":"12345abc"}' \
       http://localhost:8081/input
   ```
   Resposta esperada: `invalid zipcode` (Status 422)

3. CEP inexistente:
   ```bash
   curl -X POST \
       -H "Content-Type: application/json" \
       -d '{"cep":"00000000"}' \
       http://localhost:8081/input
   ```
   Resposta esperada: `can not find zipcode` (Status 404)

4. Testando o serviço B diretamente:
   ```bash
   curl -X GET http://localhost:8080/weather?cep=29102571
   ```

## Validando os serviços

### Verificando a disponibilidade dos serviços

1. **Serviço A**: [http://localhost:8081](http://localhost:8081)
   - Para verificar se está ativo, você pode usar:
     ```bash
     curl -I http://localhost:8081/input
     ```

2. **Serviço B**: [http://localhost:8080](http://localhost:8080)
   - Para verificar se está ativo, você pode usar:
     ```bash
     curl -I http://localhost:8080/weather
     ```

3. **Zipkin**: [http://localhost:9411](http://localhost:9411)
   - Acesse esta URL em seu navegador para visualizar a interface do Zipkin

## Resolução de problemas

- **Zipkin indisponível**: Verifique se a porta 9411 não está sendo usada por outro processo
- **Erro na API WeatherAPI**: Verifique se a chave API está correta em `docker-compose.yml`
- **Erro ao chamar o serviço B**: Verifique se os serviços estão na mesma rede do Docker
