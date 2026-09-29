# 🧱 Go Clean Architecture — Order System

Sistema de pedidos em **Go** construído com **Clean Architecture**: o mesmo caso de uso é exposto por **três interfaces** (REST, gRPC e GraphQL), com eventos publicados no **RabbitMQ**.

![Go](https://img.shields.io/badge/Go-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![gRPC](https://img.shields.io/badge/gRPC-4285F4?style=for-the-badge&logo=google&logoColor=white)
![GraphQL](https://img.shields.io/badge/GraphQL-E10098?style=for-the-badge&logo=graphql&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-005C84?style=for-the-badge&logo=mysql&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

## 🏗️ Arquitetura

```
            ┌──────────── Interfaces (infra) ────────────┐
REST :8000  │  web/order_handler.go                      │
gRPC :50051 │  grpc/service/order_service.go             │──►  UseCases  ──►  Entity (Order)
GraphQL:8080│  graph/schema.resolvers.go                 │     create_order      regras de negócio
            └────────────────────────────────────────────┘     list_orders       (FinalPrice = Price + Tax)
                                                                   │
                                     MySQL ◄── OrderRepository ◄───┤
                                  RabbitMQ ◄── OrderCreated event ◄┘
```

- `internal/entity` — entidade `Order` e suas regras, sem dependências externas
- `internal/usecase` — `CreateOrder` e `ListOrders`
- `internal/infra` — adaptadores: banco, REST, gRPC e GraphQL
- `internal/event` + `pkg/events` — dispatcher de eventos; `OrderCreated` publica no RabbitMQ
- `cmd/ordersystem` — composição das dependências com **Google Wire**

## 🚀 Como rodar

```bash
# 1. Sobe MySQL e RabbitMQ
docker compose up -d

# 2. Roda a aplicação
cd cmd/ordersystem
go run main.go wire_gen.go
```

O schema do banco está em `db/migrations`.

| Interface | Porta | Acesso |
|---|---|---|
| REST | `8000` | `POST /order` e `GET /order` (exemplos em `api/*.http`) |
| gRPC | `50051` | `OrderService.CreateOrder` e `OrderService.ListOrders` (reflection ativo) |
| GraphQL | `8080` | Playground em `http://localhost:8080` |

## 📬 Exemplos

**REST**

```http
POST http://localhost:8000/order
Content-Type: application/json

{ "id": "a", "price": 100.5, "tax": 0.5 }
```

**GraphQL**

```graphql
mutation {
  createOrder(input: { id: "b", Price: 50, Tax: 2 }) { id FinalPrice }
}

query {
  listOrders { id Price Tax FinalPrice }
}
```

**gRPC** (com [Evans](https://github.com/ktr0731/evans))

```bash
evans -r repl -p 50051
> call ListOrders
```

## 🧪 Testes

```bash
go test ./...
```

---

Desenvolvido por **Yuri Bertoldi** como desafio da pós-graduação em Go da Full Cycle — [LinkedIn](https://www.linkedin.com/in/yuri-bulh%C3%B5es-bertoldi-b62459180/)
