---
name: po-land
description: Convenção de formato de história do projeto — use ao escrever, refinar ou revisar épico, feature ou história de usuário, ao aplicar INVEST, e ao definir Definition of Ready, Definition of Done, pontos ou dependências. Também use para checar se uma história recebida está no formato aceito antes de refiná-la ou implementá-la.
---

# po-land — formato de história

Fonte de requisito: `./specs/PORTAL.md` e o card de origem. Nada além disso.
O que não estiver lá é lacuna, e lacuna se pergunta — não se preenche.

## Hierarquia

`Épico` → `Feature` → `História`. Uma história cabe em uma sprint e entrega
valor observável por si só. Se não couber, é feature e precisa ser quebrada.

## Formato da história

```
Como <persona>, quero <ação>, para <valor>
```

A persona vem do PORTAL.md; não crie persona nova. `<valor>` é o benefício, não
a repetição da ação — "para não perder a reserva" serve, "para poder reservar"
não.

## Estrutura obrigatória de cada história

```markdown
### [ID] Título curto no imperativo

**Épico:** <épico> · **Feature:** <feature>
**Card:** <ID e URL do Trello>

Como <persona>, quero <ação>, para <valor>.

#### Critérios de aceite
<Gherkin — Dado / Quando / Então. Convenção em [gherkin-land]>

#### Definition of Ready
- [ ] história no formato acima, com persona do PORTAL.md
- [ ] critérios de aceite em Gherkin, cobrindo caminho feliz e erro
- [ ] regra de negócio rastreável a uma seção do PORTAL.md
- [ ] dependências identificadas e não bloqueantes
- [ ] estimada pelo time
- [ ] impacto em dado pessoal avaliado (ver [lgpd-land])

#### Definition of Done
- [ ] critérios de aceite verdes em teste automatizado
- [ ] cenários Gherkin automatizados no E2E quando o fluxo é de ponta a ponta
- [ ] migration reversível quando houve mudança de schema
- [ ] contrato de API conforme [api-land]
- [ ] a11y verificada quando houve UI
- [ ] sem dado pessoal real em log, fixture ou seed (ver [lgpd-land])
- [ ] revisada e integrada na main

#### Estimativa
<pontos> pontos

#### Dependências
- bloqueada por: [ID]
- bloqueia: [ID]
```

## INVEST — como aplicar

| Letra | O que checar | Reprova quando |
|---|---|---|
| **I**ndependent | pode ir para a sprint sozinha | só faz sentido junto de outra |
| **N**egotiable | descreve o quê, não o como | já traz solução técnica fechada |
| **V**aluable | valor para a persona | só entrega valor interno/técnico |
| **E**stimable | o time sabe o tamanho | falta informação para estimar |
| **S**mall | cabe em uma sprint | é feature disfarçada |
| **T**estable | critério verificável | critério subjetivo ("rápido", "fácil") |

## Estimativa

Fibonacci: 1, 2, 3, 5, 8, 13. Ponto é complexidade e incerteza, não hora.
`13` é sinal de quebra, não de estimativa — quebre antes de aceitar.

## Dependências

Declare nos dois sentidos (`bloqueada por` / `bloqueia`) e sempre por ID.
Dependência circular entre histórias é erro de fatiamento: refatie.
