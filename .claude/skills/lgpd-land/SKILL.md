---
name: lgpd-land
description: Regras de dado pessoal do projeto — use ao escrever log, ao montar payload de resposta, ao criar fixture, seed, massa de teste ou tabela de Exemplos em .feature, e ao modelar coluna que guarde CPF, CNH, e-mail, telefone, endereço, cartão ou geolocalização. Também use ao revisar mensagem de erro, evento de analytics, export ou relatório, para conferir se há dado sensível exposto.
---

# lgpd-land — dado pessoal

Duas regras que não têm exceção:

1. **Dado sensível é mascarado em log e em resposta.**
2. **Nenhum dado real de pessoa em fixture, seed ou massa de teste** — inclusive
   copiado de homologação.

## O que é dado pessoal aqui

| Categoria | Campos |
|---|---|
| Identificação | CPF, RG, CNH (número, categoria, validade), passaporte |
| Contato | e-mail, telefone, endereço |
| Financeiro | cartão (PAN, CVV, validade), conta, chave Pix, renda |
| Localização | geolocalização do veículo, histórico de estação e trajeto |
| Documento | foto de CNH, selfie, comprovante de residência |

`CVV` não é armazenado, nunca — nem cifrado, nem em cache, nem em log.
Cartão: só os 4 últimos dígitos e a bandeira; o PAN completo fica no provedor
de pagamento, referenciado por token.

## Máscara — formato único

| Campo | Em log e resposta |
|---|---|
| CPF | `***.***.789-**` |
| CNH | `********901` (4 últimos) |
| E-mail | `r***@dominio.com` |
| Telefone | `(11) *****-4321` |
| Cartão | `**** **** **** 4321` |
| Endereço | logradouro e número omitidos; cidade/UF/CEP-prefixo mantidos |
| Geolocalização | precisão reduzida a cidade fora do contexto operacional |

Aplique a máscara na borda, com uma função central (`mascarar_cpf`,
`mascarar_email`, …). Máscara reimplementada por endpoint diverge em algum ponto
— e divergência aqui é vazamento.

## Log

**Nunca logue:** payload de request ou response inteiro, header `Authorization`,
cookie, token, senha, CVV, PAN, ou qualquer campo da tabela acima sem máscara.

Log estrutura e referência, não conteúdo:

```python
# não
logger.info(f"condutor cadastrado: {payload}")
# sim
logger.info("condutor cadastrado", extra={"condutor_id": id, "trace_id": tid})
```

Prefira **ID sobre valor**: `condutor_id=4821` identifica para suporte sem
expor a pessoa. O `trace_id` de [api-land] é o que liga log e requisição — é
ele que dispensa logar o payload.

Exceção que estoura a regra: mensagem de erro. `"CPF 123.456.789-00 inválido"`
vaza em log e em resposta. Escreva `"CPF inválido"` e identifique pelo campo.

## Resposta da API

O `response_model` de [api-land] é o controle primário: campo que não está no
schema de saída não vaza. Por isso, nunca retorne modelo SQLAlchemy direto.

Dado de terceiro nunca aparece: o condutor vê a própria reserva, não a de
outro. Recurso de outra pessoa é `404`, não `403` — ver [api-land].

Campo completo (CPF sem máscara, por exemplo) só em endpoint que existe para
isso, autenticado, autorizado, auditado e justificado no `./specs/PORTAL.md`.
Se o PORTAL.md não especifica quem pode ver o dado completo, **pergunte** — não
decida.

## Fixture, seed e massa de teste

Dado **sintético e reconhecível como falso**:

- CPF: gerado com dígito verificador válido, mas nunca de pessoa real; prefira
  os inválidos-por-construção (`000.000.000-00`) quando a validação permitir
- e-mail: domínio `@example.com` / `@example.org` (reservados por RFC 2606)
- telefone: prefixo `(11) 99999-` + sequência
- nome: `Condutor Teste 01`, ou nome claramente fictício
- cartão: apenas os números de teste do provedor de pagamento

Seed determinístico, versionado, sem import de dump de produção ou de
homologação. Nenhum `.csv`, `.sql` ou `.json` com dado real entra no repositório
— nem em branch, nem em anexo de card.

Em `.feature`, a tabela de `Exemplos:` segue a mesma regra: ver
[gherkin-land].

## Frontend

Máscara de exibição não é máscara de dado: o valor completo não deve chegar ao
navegador se a tela não precisa dele. Não guarde dado pessoal em
`localStorage`, `sessionStorage` ou query string — a query string entra em log
de servidor e em histórico.

Evento de analytics não carrega dado pessoal, só `condutor_id`.

## Retenção e direitos do titular

Prazo de retenção, anonimização e atendimento a pedido de exclusão ou portabilidade
saem do `./specs/PORTAL.md`. Não há default razoável para inventar aqui: se a
história toca retenção ou exclusão e o PORTAL.md é omisso, **pergunte**.

## Checklist antes de fechar

- [ ] nenhum campo da tabela aparece sem máscara em log
- [ ] `response_model` declarado, sem campo pessoal além do necessário
- [ ] mensagem de erro sem valor de dado pessoal
- [ ] fixture, seed e `Exemplos:` só com dado sintético
- [ ] nenhum dado pessoal em `localStorage` ou query string
- [ ] sem CVV e sem PAN completo em lugar algum
