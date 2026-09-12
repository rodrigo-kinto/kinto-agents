---
name: gherkin-land
description: Convenção de cenários BDD do projeto — use ao escrever ou revisar arquivo .feature, critério de aceite em Gherkin, Dado/Quando/Então, Esquema do Cenário ou Exemplos. Também use antes de automatizar um cenário, para conferir se ele está na convenção, e ao decidir o que virá teste E2E versus teste de unidade.
---

# gherkin-land — convenção de cenários

Idioma: **português**, com as palavras-chave em português do Gherkin. Linguagem
ubíqua: os termos do `./specs/PORTAL.md` e da skill `kinto-domain`, sempre os
mesmos. Nunca nome de botão, seletor, rota, tabela ou tela.

## Localização e nomenclatura

```
features/<dominio>/<fluxo>.feature
```

Um arquivo por fluxo de negócio. `<dominio>` e `<fluxo>` em kebab-case.

## Esqueleto

```gherkin
# language: pt
Funcionalidade: <valor de negócio, não a tela>
  Como <persona>, quero <ação>, para <valor>.

  Contexto:
    Dado <estado comum a todos os cenários>

  @<tag>
  Cenário: <um comportamento, afirmativo>
    Dado <estado inicial>
    Quando <uma única ação>
    Então <um resultado observável>
```

## Regras

**Um comportamento por cenário.** Se o título precisa de "e", são dois
cenários.

**Um `Quando` por cenário.** Dois `Quando` significam dois cenários — ou um
`Dado` que foi escrito como ação.

**`Então` observável pela persona.** Estado de tabela, chamada de função ou
log não são asserção de cenário; mensagem, valor em tela e resultado do fluxo
são.

**Declarativo, não imperativo.** `Quando o condutor confirma a reserva`, não
`Quando clica em "Confirmar" e preenche o CPF`. O como vive no step, não no
cenário.

**Sem encadeamento entre cenários.** Cada cenário parte do `Contexto` e é
independente. Ordem de execução nunca é premissa.

**`E` / `Mas` para continuar**, nunca para iniciar um bloco.

## Dados variáveis

Use `Esquema do Cenário:` quando a mesma regra muda só por dado:

```gherkin
  Esquema do Cenário: bloqueio por idade mínima do condutor
    Dado um condutor com <idade> anos
    Quando solicita a reserva da categoria "<categoria>"
    Então a reserva é <resultado>

    Exemplos:
      | idade | categoria | resultado |
      | 20    | compacto  | recusada  |
      | 25    | compacto  | aceita    |
```

Os valores da tabela vêm do PORTAL.md. Se a faixa não está especificada,
pergunte — não escolha um número.

## Cobertura mínima por história

- caminho feliz
- cada regra de negócio, incluindo a borda de cada limite
- cada mensagem de erro especificada
- concorrência quando a regra é de exclusividade — reserva sobreposta, último
  veículo disponível

## Tags

| Tag | Uso |
|---|---|
| `@e2e` | vai para a suíte Playwright |
| `@critico` | cadastro do condutor, reserva, checkout — sempre `@e2e` |
| `@wip` | incompleto, fora da suíte de CI |

## Dado pessoal em cenário

Nome, CPF, CNH, e-mail e telefone em `Exemplos:` são **fictícios e
reconhecíveis como tal**. Ver [lgpd-land]. Nunca dado de pessoa real, mesmo de
ambiente de homologação.

## Fora de escopo

O `.feature` especifica; não automatiza. A implementação dos steps é do
`playwright-tester`. Cenário que não descreve comportamento de persona é teste
de unidade e não pertence aqui.
