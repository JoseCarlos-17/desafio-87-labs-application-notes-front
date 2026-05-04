# desafio-87-blue-application-notes-front

Interface web do desafio técnico **desafio-87-blue-application-notes-front** em **Vue 3**, com **Composition API** e `<script setup>`. O aplicativo consome a API REST do repositório **`desafio-87-blue-application-notes-api`** para:

- **criar** notas (formulário com tratamento de **`422`** retornado pela API);
- **listar** notas em tabela com **paginação** (Previous / Next e “Page X of Y”);
- **filtrar** por **título**, enviando o termo como query string em **`GET /notes`**.

A camada HTTP usa **Axios**, centralizada em **`src/axios.js`**, com **`baseURL`** apontando para a API (por padrão **`http://localhost:3000`**). O atributo `created_at` das notas criadas é formatado com **date-fns**.

---

## Stack principal

| Item | Uso |
|------|-----|
| Vue 3 | Interface e reatividade |
| Axios | Cliente HTTP |
| date-fns | Formatação de datas |

---

## Pré-requisitos

- **Node.js** (LTS recomendado).
- O repositório inclui **`yarn.lock`** — recomenda-se **`yarn install`**.
- A API deve estar acessível na URL configurada em **`src/axios.js`** (padrão **`http://localhost:3000`**). O Vue CLI costuma servir o front em **`http://localhost:8080`**; a API deve permitir essa origem em **CORS**.

---

## Instalação de dependências

Na raiz do projeto:

```bash
yarn install
```

## Execução

| Comando | Descrição |
|---------|-----------|
| `yarn serve` | Servidor de desenvolvimento com hot-reload (porta padrão **8080**; outra porta se 8080 estiver ocupada). |

Com npm:

```bash
npm run serve
```

---

## Endpoints HTTP utilizados

Caminhos são relativos à **`baseURL`** que consta no arquivo "src/axios.js", `http://localhost:3000`. JSON em **snake_case**, alinhado à API Rails.

### `GET /notes`

Listagem paginada; filtro opcional por título.

| Campo | Valor |
|--------|--------|
| **Método** | `GET` |
| **Path** | `/notes` |

**Query string** (como o front envia):

| Parâmetro | Obrigatório | Descrição |
|-----------|-------------|-----------|
| `page` | Sim (no cliente) | Página atual (≥ 1). |
| `limit` | Sim (no cliente) | Itens por página (ex.: **10**). |
| `title` | Não | Trecho do título; se ausente ou só espaços, não é enviado e a API lista todas as notas (com paginação). |

**Respostas**

| Status code | Response body (JSON) |
|-------------|----------------------|
| **200 OK** | Objeto com listagem e metadados de paginação. |

Exemplo de **`response_body`** em sucesso:

```json
{
  "notes": [
    {
      "id": 1,
      "title": "Minha nota",
      "content": "Conteúdo",
      "created_at": "2026-05-03T12:00:00.000Z"
    }
  ],
  "total_pages": 5,
  "current_page": 1
}
```

Erros de servidor ou indisponibilidade da API aparecem como falha de rede / status **5xx** no Axios (sem formato fixo no front).

---

### `POST /notes`

Criação de nota.

| Campo | Valor |
|--------|--------|
| **Método** | `POST` |
| **Path** | `/notes` |

**Request body** (JSON):

```json
{
  "note": {
    "title": "string",
    "content": "string"
  }
}
```

**Respostas**

| Status code | Response body (JSON) | Uso no front |
|-------------|------------------------|--------------|
| **201 Created** | Objeto da nota: `id`, `title`, `content`, `created_at`. | Recarrega a lista (volta à página **1**), limpa o formulário e limpa erros. |
| **422 Unprocessable Entity** | `{ "error": "mensagem de validação (ex.: título fora de 5–50 caracteres)" }` | Mensagem exibida ao usuário (área de erro do formulário). |

Exemplo de **`response_body`** em **201**:

```json
{
  "id": 42,
  "title": "Nova nota",
  "content": "Texto",
  "created_at": "2026-05-03T14:30:00.000Z"
}
```

Exemplo de **`response_body`** em **422**:

```json
{
  "error": "Validation failed: Title is too short (minimum is 5 characters)"
}
```

caso tente enviar uma nota sem título:

```json
{
  "error": "Title can't be blank"
}
```

---

## Ausência de TypeScript e de testes com Vue Test Utils

Por **limitação de tempo** no escopo do desafio, o código permanece em **JavaScript** sem **TypeScript**. Adotar TS exigiria `tsconfig`, tipagem de props/emits, contratos com a API e possivelmente sincronização com o back-end — esforço não priorizado frente às entregas de CRUD, paginação e filtro.

Da mesma forma, **não há testes de componentes** com **Vue Test Utils** (nem suíte Vitest/Jest específica para o front). Testar formulário, tabela, paginação e chamadas Axios assíncronas implica mocks de HTTP, montagem de SFCs e manutenção de fixtures. A confiança ficou nos **testes da API (RSpec)** e na **validação manual** no navegador.
