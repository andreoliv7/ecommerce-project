# Vero Mercado — Marketplace Full-Stack

Plataforma de e-commerce completa construída para demonstrar fluxo comercial de
ponta a ponta: catálogo, autenticação com controle de acesso por papel,
carrinho persistente, checkout com pagamento simulado e um painel
administrativo.

## Stack

| Camada       | Tecnologia |
|--------------|------------|
| Front-end    | Next.js 14 (App Router) + TypeScript + Tailwind CSS |
| Back-end     | Node.js + Express + TypeScript |
| Banco de dados | SQLite via Prisma ORM (zero-config; troque para PostgreSQL em 1 linha — veja abaixo) |
| Autenticação | JWT + bcrypt, com RBAC (`USER` / `ADMIN`) |
| Estado do carrinho | Zustand com persistência em `localStorage` |
| Pagamento    | Gateway simulado (latência artificial + regra de recusa), no estilo Stripe em modo teste |

## Estrutura

```
ecommerce-project/
  backend/     API REST (Express + Prisma)
  frontend/    Aplicação Next.js
```

## Como rodar localmente

### 1. Back-end

```bash
cd backend
cp .env.example .env
npm install
npx prisma migrate dev --name init   # cria o banco SQLite e as tabelas
npm run seed                          # popular com categorias, produtos e usuários de teste
npm run dev                           # sobe em http://localhost:4000
```

Usuários criados pelo seed:
- **Admin:** `admin@marketplace.com` / `admin123`
- **Cliente:** `user@marketplace.com` / `user123`

### 2. Front-end

Em outro terminal:

```bash
cd frontend
cp .env.local.example .env.local
npm install
npm run dev                           # sobe em http://localhost:3000
```

Abra `http://localhost:3000`. Faça login como admin para acessar `/admin` e
cadastrar/editar produtos e categorias, ou como cliente para navegar, comprar
e ver o histórico de pedidos em `/orders`.

## Testando o pagamento simulado

Na tela de checkout, qualquer número de cartão funciona. Para simular uma
**recusa do gateway**, use um número terminado em `0000` (ex.: `4000 0000 0000
0000`). Nesse caso o pedido é marcado como `FAILED` e o estoque reservado é
devolvido automaticamente — o mesmo fluxo que um gateway real dispararia.

## Trocando para PostgreSQL ou MySQL

O SQLite é usado por padrão para rodar sem infraestrutura extra. Para usar
Postgres:

1. Suba um Postgres (local, Docker ou serviço gerenciado).
2. Em `backend/prisma/schema.prisma`, troque `provider = "sqlite"` por
   `provider = "postgresql"`.
3. Em `backend/.env`, aponte `DATABASE_URL` para a string de conexão do
   Postgres, ex.: `postgresql://user:senha@localhost:5432/marketplace`.
4. Rode `npx prisma migrate dev --name init` novamente.

Para MySQL o processo é o mesmo, trocando o provider para `"mysql"`.

## O que foi implementado (mapeado ao briefing)

- **Autenticação e Autorização:** registro/login com JWT (`bcryptjs` para
  hashing), middleware `authenticate` + `authorize(...roles)` para RBAC real
  entre usuário comum e administrador.
- **Modelagem relacional:** tabelas `User`, `Category`, `Product`, `Order`,
  `OrderItem` e `Payment`, com relações 1:N e N:N via `OrderItem` (ver
  `backend/prisma/schema.prisma`).
- **Integração com API externa (simulada):** `simulatePaymentGateway()` em
  `backend/src/routes/order.routes.ts` reproduz uma chamada assíncrona a um
  gateway de pagamento (latência de rede, transaction ID, sucesso/recusa),
  com efeitos reais no banco (baixa/estorno de estoque, status do pedido).
- **Gerenciamento de estado:** carrinho de compras persistente com Zustand +
  `persist` (localStorage), sobrevive a recarregamentos de página.

## Limitações conhecidas (é um projeto de demonstração)

- O "gateway de pagamento" é uma simulação local — nenhuma chave real ou
  chamada externa é feita.
- Não há upload de imagens: produtos usam URLs de imagem.
- Sem testes automatizados incluídos (é um bom próximo passo: Jest/Vitest no
  backend, Playwright para e2e).
