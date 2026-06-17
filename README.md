# EduReach (code-backed summary)

Published: https://edureach-agentic-chatbot.onrender.com/

## Overview
- Purpose: Web application that provides an authenticated SPA and a knowledge-grounded conversational assistant backed by a retrieval pipeline.
- Target users: end-users of the SPA (frontend code includes login/signup and chat UI).

## Key Features (code-verified)
- JWT-based user registration, login, and profile (`server/src/controllers/auth.controller.ts`).
- Client-side authentication flow and token storage (`client/src/context/AuthContext.tsx`, `client/src/services/auth.service.ts`).
- Chat endpoint that returns RAG-generated responses (`server/src/controllers/chat.controller.ts`, `server/src/services/rag.service.ts`).
- Knowledge-base ingestion and vector indexing from `server/knowledge-base/edureach-knowledge.txt` (`server/src/services/rag.service.ts`).
- Outbound calling integration via Vapi (`server/src/services/vapi.service.ts`) and protected call route (`server/src/routes/vapi.routes.ts`).

## Tech Stack (explicit)
- Frontend: React + TypeScript, Vite (`client/`)
- Backend: Node.js + TypeScript, Express-style routing (`server/src/app.ts`)
- Database: MongoDB (used via Mongoose models and native MongoDB client) (`server/src/config/database.config.ts`, `server/src/models`)
- Authentication: JWT (`server/src/utils/jwt.util.ts`, `server/src/middleware/auth.middleware.ts`)
- AI/ML: LangChain + Google GenAI embeddings and Chat model; MongoDB Atlas vector search via LangChain (`server/src/services/rag.service.ts`)

## Architecture (what the code implements)
- Request flow (code evidence):
  - Frontend calls REST endpoints (client services: `client/src/services/*.ts`).
  - Server mounts routes at `/api/auth`, `/api/chat`, `/api/vapi` (`server/src/app.ts`).
  - Auth routes use JWT middleware; protected routes require `auth.middleware` (`server/src/routes/auth.routes.ts`, `server/src/middleware/auth.middleware.ts`).
  - Chat controller calls `getRAGResponse` which uses LangChain embeddings, vector store, and a chat model (`server/src/controllers/chat.controller.ts`, `server/src/services/rag.service.ts`).

## Project Structure
- `client/` — React app: `src/components`, `src/pages`, `src/services` (API clients).
- `server/` — Express-like API: `src/controllers`, `src/routes`, `src/services`, `src/models`, `src/middleware`, `src/config`.
- `server/knowledge-base/edureach-knowledge.txt` — plaintext source loaded by `rag.service`.

## Setup & Installation (from code)
### Prerequisites (implied by package.json & code)
- Node.js, npm
- A MongoDB URI available in `MONGODB_URI` for `server/src/config/database.config.ts`.
- `GOOGLE_API_KEY` for embeddings and chat in `server/src/services/rag.service.ts`.
- `JWT_SECRET` for `server/src/utils/jwt.util.ts`.
- Vapi credentials (`VAPI_API_KEY`, `VAPI_PHONE_NUMBER_ID`, `VAPI_ASSISTANT_ID`) for `server/src/services/vapi.service.ts` (if using call feature).

### Install
1. Server: `cd server && npm install`
2. Client: `cd client && npm install`

### Run (development)
- Backend dev script: `npm run dev` in `server/` (`server/package.json`).
- Frontend dev script: `npm run dev` in `client/` (`client/package.json`).

Note: client `API` base is `http://localhost:5001/api` (`client/src/services/api.ts`). Server default port in code is `5002` (`server/src/server.ts`). These values are explicit in source and may require alignment via env vars or client config.

## API Endpoints (discovered in code)
- `POST /api/auth/register` — `server/src/routes/auth.routes.ts` -> `server/src/controllers/auth.controller.ts` (creates user, returns JWT).
- `POST /api/auth/login` — `server/src/routes/auth.routes.ts` -> `server/src/controllers/auth.controller.ts` (validates credentials, returns JWT).
- `GET /api/auth/me` — protected route using `auth.middleware` (`server/src/routes/auth.routes.ts`).
- `POST /api/chat/message` — `server/src/routes/chat.routes.ts` -> `server/src/controllers/chat.controller.ts` (calls RAG service).
- `POST /api/vapi/call` — `server/src/routes/vapi.routes.ts` -> `server/src/controllers/vapi.controller.ts` (protected route; calls `server/src/services/vapi.service.ts`).

## AI / RAG Workflow (implemented in code)
- Document ingestion: `server/src/services/rag.service.ts` loads `server/knowledge-base/edureach-knowledge.txt` via `TextLoader` and splits text (`RecursiveCharacterTextSplitter`).
- Embeddings: created using `GoogleGenerativeAIEmbeddings` (requires `GOOGLE_API_KEY`) (`server/src/services/rag.service.ts`).
- Vector store: `MongoDBAtlasVectorSearch` is used to store and query embeddings in collection `knowledge_docs` (`server/src/services/rag.service.ts`).
- Response generation: `createAgent` with `ChatGoogleGenerativeAI` invokes the retrieve tool and generates final answer (`server/src/services/rag.service.ts`).

## Security (code-evident)
- JWT generation and verification: `server/src/utils/jwt.util.ts`.
- Auth enforcement middleware: `server/src/middleware/auth.middleware.ts` (applied to protected routes in route files).
- Password hashing: `server/src/utils/password.util.ts` (used in `auth.controller.ts`).

## Challenges Observed (from code)
- Mismatch between client API base (`client/src/services/api.ts`) and server default port (`server/src/server.ts`) — requires explicit configuration before running.
- RAG pipeline requires `GOOGLE_API_KEY` and MongoDB vector index configuration; `rag.service` logs steps and will fail without these.

## Future Enhancements (code-aligned suggestions)
- Align client `baseURL` with server `PORT` or make the client use an env-configurable base URL (`client/src/services/api.ts`).
- Add explicit vector index creation script or documentation because `rag.service` logs an Atlas index requirement (`server/src/services/rag.service.ts`).
- Add unit/integration tests for controllers and services (no tests present in repo).

## Short Explanations (concise, code-backed)
1) 30-second summary
- EduReach is a TypeScript React SPA + Node API. It authenticates users with JWT and provides a chat endpoint that returns knowledge-grounded answers using LangChain + Google GenAI embeddings stored in MongoDB (`server/src/services/rag.service.ts`).

2) 1-minute explanation
- The client stores tokens in localStorage and calls server endpoints (`client/src/services/*`). The server offers auth, chat, and a protected outbound-call route. The chat flow uses `rag.service` to load and embed documents, store them in a Mongo-backed vector store, and run a LangChain agent with a chat model to produce answers (`server/src/controllers/chat.controller.ts`, `server/src/services/rag.service.ts`).

3) 3-minute explanation
- Walk through code: signup/login handled in `server/src/controllers/auth.controller.ts` with password hashing (`server/src/utils/password.util.ts`) and token generation (`server/src/utils/jwt.util.ts`). The frontend `AuthContext` stores the token and fetches `/auth/me` (`client/src/context/AuthContext.tsx`). The chat UI (`client/src/components/ChatDrawer.tsx`) posts messages to `/api/chat/message`, which invokes `getRAGResponse` in `server/src/services/rag.service.ts`. That service uses `TextLoader` to load `server/knowledge-base/edureach-knowledge.txt`, splits text, embeds chunks via `GoogleGenerativeAIEmbeddings`, stores them in MongoDB via `MongoDBAtlasVectorSearch`, and constructs a LangChain agent (`createAgent`) that uses a `retrieve` tool and `ChatGoogleGenerativeAI` to answer queries.

4) Common interviewer questions & concise answers (code-backed)
- Q: Where are embeddings produced? A: `server/src/services/rag.service.ts` uses `GoogleGenerativeAIEmbeddings` (requires `GOOGLE_API_KEY`).
- Q: How is auth enforced? A: `server/src/middleware/auth.middleware.ts` verifies JWT (`server/src/utils/jwt.util.ts`) and is used on protected routes like `/api/auth/me` and `/api/vapi/call`.
- Q: What stores the knowledge vectors? A: MongoDB collection `knowledge_docs` via `MongoDBAtlasVectorSearch` in `server/src/services/rag.service.ts`.

---

## Verification Table
| README Statement | Evidence File(s) | Verified |
|---|---|---:|
| JWT auth (register/login/me) | server/src/controllers/auth.controller.ts, server/src/routes/auth.routes.ts | Yes |
| Client stores token and calls `/auth/me` | client/src/context/AuthContext.tsx, client/src/services/auth.service.ts | Yes |
| Chat endpoint backed by RAG | server/src/routes/chat.routes.ts, server/src/controllers/chat.controller.ts | Yes |
| RAG implementation uses Google embeddings and LangChain | server/src/services/rag.service.ts | Yes |
| Knowledge base file is `server/knowledge-base/edureach-knowledge.txt` | server/knowledge-base/edureach-knowledge.txt, server/src/services/rag.service.ts | Yes |
| MongoDB connection required | server/src/config/database.config.ts, server/src/services/rag.service.ts | Yes |
| Vapi outbound call integration and protected route | server/src/services/vapi.service.ts, server/src/routes/vapi.routes.ts, server/src/controllers/vapi.controller.ts | Yes |
| Client API base URL is `http://localhost:5001/api` | client/src/services/api.ts | Yes |
| Server default port is `5002` in code | server/src/server.ts | Yes |
| No deployment/Docker/CI config found | repository search (no Dockerfile/.github/etc) | Yes |



