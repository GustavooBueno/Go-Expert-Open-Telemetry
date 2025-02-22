# Projeto clima por CEP (OTEL e Zipkin)

Este projeto é composto por dois serviços (A e B) que trabalham em conjunto para fornecer informações de temperatura com base em um CEP, além de integrar OpenTelemetry (OTEL) e Zipkin para observabilidade e tracing distribuído.

Serviço A: Recebe um CEP via requisição POST (JSON), verifica se possui um formato válido (8 dígitos) e, caso positivo, encaminha a solicitação para o Serviço B.

Serviço B: Processa o CEP recebido, obtém a cidade correspondente através do ViaCEP, recupera a temperatura atual via WeatherAPI, converte os valores para Fahrenheit e Kelvin e retorna o resultado junto com o nome da cidade.

Ambos os serviços geram spans de tracing e enviam para um Otel Collector, que exporta os dados para o Zipkin, possibilitando a análise dos traces e spans.

## Requisitos

- Docker
- Docker Compose

## Iniciar o projeto

1. Clone o repositório e entre no diretório do projeto:
   ```bash
   git clone https://github.com/GustavooBueno/Go-Expert-Open-Telemetry.git
   cd Go-Expert-Open-Telemetry
   ```
2. Ajuste a variável `WEATHER_API_KEY` no arquivo `docker-compose.yml` com a sua chave da [WeatherAPI](https://www.weatherapi.com/).

3. Inicie a aplicação e as ferramentas de observabilidade:

   ```bash
   docker-compose up --build

   ```

4. Testando a aplicação:,

   ```bash
   curl -X POST \
       -H "Content-Type: application/json" \
       -d "{\"cep\":\"29102571\"}" \
       http://localhost:8081/input

   ```

5. Serviços:

   - **Serviço A**: [http://localhost:8081](http://localhost:8081)
   - **Serviço B**: [http://localhost:8080](http://localhost:8080)

   - **Zipkin**: [http://localhost:9411](http://localhost:9411)  
     Acesse esta URL para visualizar os traces coletados.
