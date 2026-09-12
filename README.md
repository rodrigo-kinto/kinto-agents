# kinto-agents

Plugin do [Claude Code](https://claude.com/claude-code) com o time de
sub-agentes e as convenções do portal de locação e venda de veículos da Kinto.

A ideia é simples: **você fala com um agente só.** O `orquestrador` é a porta de
entrada — ele classifica a tarefa, escolhe os especialistas, executa em série ou
em paralelo conforme a dependência, valida a saída contra os critérios de aceite
e devolve um relatório de uma linha por agente acionado.

Toda regra de negócio vem de `./specs/PORTAL.md`. O que não estiver lá, nenhum
agente inventa — ele pergunta.

## Instalação

### Como plugin de usuário (recomendado)

Clone dentro do diretório de skills do Claude Code. O plugin é detectado
automaticamente na próxima sessão:

```bash
git clone https://github.com/rodrigo-kinto/kinto-agents.git \
  ~/.claude/skills/kinto-agents
```

Para carregar sem reiniciar, rode `/reload-plugins` na sessão aberta.

Confira se subiu com os 9 agentes e as 4 skills:

```bash
claude plugin list
claude plugin details kinto-agents
```

### Só nesta sessão

Sem instalar nada, aponte para um clone em qualquer lugar:

```bash
git clone https://github.com/rodrigo-kinto/kinto-agents.git
claude --plugin-dir ./kinto-agents
```

### Gerenciando

```bash
claude plugin disable kinto-agents@skills-dir   # desliga
claude plugin enable  kinto-agents@skills-dir   # liga
rm -rf ~/.claude/skills/kinto-agents            # remove
```

## Os agentes

Chame o `orquestrador` e deixe que ele distribua. Os demais existem para serem
acionados por ele, não por você.

| Agente | O que faz |
|---|---|
| **orquestrador** | Porta de entrada. Classifica a tarefa (história, bug, spike, refactor ou infra), escolhe os especialistas, encadeia o que depende e paraleliza o que não depende, valida a entrega contra os critérios de aceite e reporta. Não escreve código. |
| **product-owner** | Transforma o `PORTAL.md` em épicos, features e histórias no formato "Como &lt;persona&gt;, quero &lt;ação&gt;, para &lt;valor&gt;". Aplica INVEST e entrega critérios de aceite em Gherkin, Definition of Ready, Definition of Done, pontos e dependências. |
| **tech-lead** | Refina a história tecnicamente e quebra em subtarefas por trilha — backend, frontend, banco e QA — com dependências, ordem de execução, o que pode ir em paralelo e os riscos a decidir antes. |
| **user-story-reader** | Lê o card no Trello via MCP e devolve a história estruturada: transcreve título, descrição, checklists e comentários, e lista as lacunas em vez de preenchê-las. |
| **bdd-gherkin** | Converte os critérios de aceite em arquivos `.feature`: caminho feliz, borda de cada limite, mensagens de erro e concorrência. Especifica; não automatiza. |
| **playwright-tester** | Testes E2E com Playwright em TypeScript, Page Objects e web-first assertions. Cobre obrigatoriamente **cadastro do condutor, reserva e checkout**. Reporta o defeito em vez de adaptar o produto ao teste. |
| **frontend-react** | UI em React 18 + Vite + TypeScript, React Router e estado com hooks. Acessibilidade e performance como requisito, seguindo `./specs/design-tokens.json` — nenhum valor de cor ou espaçamento hard-coded. |
| **backend-python** | Dono da API: FastAPI, Pydantic, SQLAlchemy e Alembic. Regra de negócio em camada de serviço, nunca no roteador. Toda mudança de schema entra por migration reversível. |
| **db-postgres** | Modelagem, migrations e seed no PostgreSQL. Garante **no banco** que disponibilidade por estação e período não permita reserva sobreposta — `EXCLUDE USING gist` sobre `tstzrange`, não checagem em aplicação sujeita a corrida. |

## As skills

Convenções carregadas sob demanda, quando o assunto aparece. Os agentes as
consultam sozinhos.

| Skill | Assunto |
|---|---|
| **po-land** | Formato de história: hierarquia épico/feature/história, estrutura obrigatória, checklist INVEST, escala de pontos e como declarar dependência. |
| **gherkin-land** | Convenção de cenários: um comportamento e um `Quando` por cenário, `Então` observável pela persona, linguagem declarativa, `Esquema do Cenário` para dado variável, tags e cobertura mínima. |
| **api-land** | Padrão de rota e de erro: `/api/v1/<recurso>`, método e status por caso, schemas Pydantic por papel, paginação e um único formato de erro com `codigo` estável e `trace_id`. |
| **lgpd-land** | Dado pessoal: o que mascarar e em que formato, o que nunca logar, `response_model` como controle de vazamento e dado sintético obrigatório em fixture, seed e `Exemplos:`. |

## Estrutura

```
.claude-plugin/plugin.json   manifesto do plugin
agents/                      os 9 sub-agentes
skills/                      as 4 skills
.claude/agents -> ../agents  symlink, para os agentes valerem também neste repo
.claude/skills -> ../skills
```

Os arquivos reais ficam em `agents/` e `skills/` na raiz, onde o carregador de
plugins os descobre. Os symlinks em `.claude/` fazem o repositório usar o
próprio time enquanto você o desenvolve.

> Em Windows sem symlink habilitado no Git (`core.symlinks=true` ou modo
> desenvolvedor), os links de `.claude/` são baixados como arquivo de texto. O
> plugin continua funcionando — só o uso local dentro deste repositório deixa de
> valer.

## Pré-requisitos ainda não presentes

Os agentes referenciam três coisas que este repositório ainda não contém:

- `./specs/PORTAL.md` — fonte de toda regra de negócio
- `./specs/design-tokens.json` — tokens seguidos pelo `frontend-react`
- skill `kinto-domain` — vocabulário do domínio

Até existirem, os agentes vão pedir a informação em vez de assumir.

O `orquestrador` e o `user-story-reader` também dependem de servidores MCP de
Trello e GitHub configurados na sua instalação do Claude Code.

## Segurança

Nenhuma credencial, token ou dado pessoal é versionado aqui — o `.gitignore`
cobre `.env`, chaves e `/scan/raw/`, e a skill `lgpd-land` proíbe dado real em
fixture e seed. Este repositório é público: confira isso antes de cada push.
