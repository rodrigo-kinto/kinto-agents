---
name: api-land
description: Padrão de rota e de erro da API do projeto — use ao criar ou revisar endpoint FastAPI, ao nomear rota ou recurso, ao escolher método e status code, ao montar schema Pydantic de request/response, ao paginar, versionar ou tratar erro de domínio. Também use ao consumir a API no frontend, para saber o contrato e o formato de erro esperados.
---

# api-land — padrão de rota e erro

Stack: FastAPI + Pydantic + SQLAlchemy. Regra de negócio em camada de serviço;
o roteador só traduz HTTP ↔ domínio.

## Rota

```
/api/v1/<recurso-no-plural>
```

- recurso no **plural**, kebab-case: `/api/v1/condutores`, `/api/v1/reservas`
- substantivo, nunca verbo: `POST /reservas`, não `/criarReserva`
- aninhe só um nível, e só quando o filho não existe sem o pai:
  `/reservas/{reserva_id}/pagamentos`
- filtro em query string, não em caminho:
  `GET /veiculos?estacao=SP01&inicio=…&fim=…`
- ação que não é CRUD vira sub-recurso no imperativo:
  `POST /reservas/{id}/cancelamento`
- versão no caminho (`/v1`); quebra de contrato exige `/v2`, nunca alteração
  de `/v1`

Um `APIRouter` por domínio, com `prefix` e `tags`. Sessão e usuário autenticado
sempre por `Depends`.

## Método e status

| Método | Uso | Sucesso |
|---|---|---|
| `GET` | leitura, sem efeito | `200` |
| `POST` | cria recurso | `201` + `Location` |
| `POST` | executa ação (sub-recurso) | `200` ou `202` |
| `PUT` | substitui por completo | `200` |
| `PATCH` | altera parcialmente | `200` |
| `DELETE` | remove | `204`, sem corpo |

Coleção vazia é `200` com lista vazia, nunca `404`.

## Schemas Pydantic

Três schemas por recurso, nomeados pelo papel:

```python
class ReservaCreate(BaseModel):   # entrada
class ReservaUpdate(BaseModel):   # entrada parcial, campos opcionais
class ReservaRead(BaseModel):     # saída
```

- `ReservaRead` é o único contrato de saída — nunca retorne modelo SQLAlchemy
- campos em `snake_case`; datas em ISO-8601 com timezone (`datetime`, tz-aware)
- dinheiro em `Decimal` com moeda explícita, nunca `float`
- `model_config = ConfigDict(extra="forbid")` na entrada: campo desconhecido é
  erro, não é ignorado
- `response_model` declarado em todo endpoint — é o que impede vazamento de
  campo (ver [lgpd-land])

## Paginação

Toda coleção pagina, desde o primeiro dia:

```
GET /api/v1/reservas?limit=20&offset=0
```

`limit` default 20, máximo 100. Resposta:

```json
{ "items": [], "total": 0, "limit": 20, "offset": 0 }
```

## Erro

Um único formato, em toda a API:

```json
{
  "erro": {
    "codigo": "RESERVA_SOBREPOSTA",
    "mensagem": "Já existe reserva para este veículo no período informado.",
    "detalhes": [
      { "campo": "periodo_fim", "mensagem": "sobrepõe a reserva 4821" }
    ],
    "trace_id": "01J8…"
  }
}
```

- `codigo`: `SCREAMING_SNAKE_CASE`, estável — é contrato, o frontend ramifica
  nele. `mensagem` é texto de UI e pode mudar.
- `detalhes`: só para erro de validação por campo; omitido nos demais.
- `trace_id`: sempre presente, correlaciona com o log.
- `mensagem` nunca expõe stack trace, SQL, nome de tabela ou dado pessoal de
  terceiro.

### Status por classe de erro

| Status | Quando | Código exemplo |
|---|---|---|
| `400` | requisição malformada | `PAYLOAD_INVALIDO` |
| `401` | sem autenticação ou token inválido | `NAO_AUTENTICADO` |
| `403` | autenticado, sem permissão | `ACESSO_NEGADO` |
| `404` | recurso inexistente ou não visível ao solicitante | `RESERVA_NAO_ENCONTRADA` |
| `409` | conflito com o estado atual | `RESERVA_SOBREPOSTA` |
| `422` | validação de campo | `VALIDACAO` |
| `429` | limite de requisições | `LIMITE_EXCEDIDO` |
| `500` | falha não prevista | `ERRO_INTERNO` |

`404` em vez de `403` quando revelar a existência do recurso já é vazamento —
reserva de outro condutor, por exemplo.

Recurso que existe mas não pertence ao solicitante nunca retorna seu conteúdo.

### Implementação

Erro de domínio é exceção da camada de serviço, traduzida por
`exception_handler` no app — nunca `HTTPException` solta no meio da regra:

```python
class ErroDominio(Exception):
    codigo: str
    status: int
```

Um handler para `ErroDominio`, um para `RequestValidationError` (que remapeia o
formato do FastAPI para o acima) e um para `Exception` — este último loga com
`trace_id` e responde `ERRO_INTERNO`, sem detalhe interno no corpo.

## Idempotência

`POST` que movimenta dinheiro ou cria reserva aceita `Idempotency-Key` no
header e repete a resposta original para a mesma chave.

## Fora de escopo

A invariante de reserva sobreposta é garantida no banco, não aqui — a API
traduz a violação em `409 RESERVA_SOBREPOSTA`. Ver o agente `db-postgres`.
