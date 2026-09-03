## Ideia

O projeto é um avaliador de currículos e perfis do LinkedIn. A aplicação recebe os dados profissionais de uma pessoa, processa essas informações e apresenta uma avaliação com pontos fortes, oportunidades de melhoria e recomendações para tornar o perfil mais relevante.

O sistema é dividido em duas partes principais:

- **Core**: aplicação web em Laravel. É responsável pela interface, rotas, autenticação, persistência, criação dos pedidos de avaliação e comunicação com os serviços da aplicação.
- **Worker**: serviço em .NET responsável pelo processamento assíncrono das avaliações. Ele deverá consumir tarefas, analisar os dados de CV/LinkedIn e devolver os resultados ao core.

PostgreSQL com pgvector será usado como banco de dados e RabbitMQ como mensageria entre o core e o worker.

## Arquitetura

```text
Usuário
   |
   v
Nginx :8000
   |
   v
Core (Laravel/PHP-FPM) ----> PostgreSQL + pgvector
   |
   v
RabbitMQ <------------------ Worker (.NET)
```

Todos os serviços são executados pela mesma rede Docker Compose. Os volumes `postgres_data` e `rabbitmq_data` preservam os dados entre reinicializações.

## Estado atual

O projeto está na base inicial de desenvolvimento:

- O core é uma aplicação Laravel 13 rodando em PHP 8.3.
- O worker é um `BackgroundService` em .NET 10 e, por enquanto, apenas registra uma mensagem periodicamente.
- A conexão efetiva entre o core, o RabbitMQ e o worker ainda precisa ser implementada.
- A página inicial atual ainda é a tela padrão do Laravel.

## Pré-requisitos

- Docker
- Docker Compose
- Git

Não é necessário instalar PHP, Composer ou .NET localmente para executar os serviços pelo Compose.

## Como rodar

O arquivo de ambiente do projeto fica dentro de `core/`. Na raiz do projeto, copie o exemplo:

```bash
cp core/.env.example core/.env
```

Edite `core/.env` e configure pelo menos estas variáveis:

```dotenv
APP_NAME=CVLN-AI
APP_URL=http://localhost:8000
APP_KEY=

DB_CONNECTION=pgsql
DB_HOST=pgvector
DB_PORT=5432
DB_DATABASE=db_main
DB_USERNAME=main
DB_PASSWORD=password
```

O Docker Compose também usa `core/.env` para o core, o worker e o PostgreSQL. Depois, suba os containers:

```bash
docker compose up --build -d
```

Gere a chave da aplicação dentro do container do core. O valor será gravado em `core/.env`:

```bash
docker compose exec core php artisan key:generate
```

Por fim, execute as migrations:

```bash
docker compose exec core php artisan migrate
```

A aplicação estará disponível em [http://localhost:8000](http://localhost:8000).

## Serviços locais

| Serviço | Endereço | Responsabilidade |
| --- | --- | --- |
| Aplicação | [http://localhost:8000](http://localhost:8000) | Interface e API do core |
| PostgreSQL | `localhost:5432` | Banco de dados com pgvector |
| RabbitMQ AMQP | `localhost:5672` | Comunicação entre core e worker |
| RabbitMQ Management | [http://localhost:15672](http://localhost:15672) | Administração das filas |

As credenciais do RabbitMQ são definidas pelas variáveis `RABBITMQ_DEFAULT_USER` e `RABBITMQ_DEFAULT_PASS`, caso sejam configuradas no `core/.env`.

## Comandos úteis

Ver logs de todos os serviços:

```bash
docker compose logs -f
```

Ver apenas os logs do core ou do worker:

```bash
docker compose logs -f core
docker compose logs -f worker
```

Parar os serviços sem remover os dados:

```bash
docker compose down
```

Parar os serviços e remover os volumes persistentes:

```bash
docker compose down -v
```

Executar os testes do core:

```bash
docker compose exec core php artisan test
```

## Estrutura do projeto

```text
.
├── core/       # Aplicação web Laravel
├── worker/     # Processador assíncrono em .NET
└── docker-compose.yml
```

## Desenvolvimento

O `core/` e o `worker/` são montados como volumes nos respectivos containers. Alterações no código local ficam disponíveis nos serviços sem precisar recriar os containers. Quando houver mudanças em dependências ou nos Dockerfiles, refaça a imagem:

```bash
docker compose up --build -d
```