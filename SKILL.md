# Guia de Avaliação Técnica — Desenvolvedor(a) Full Stack

> **Uso interno — Academia do Universitário**
> Este documento é um roteiro estruturado para que o avaliador humano consiga, a partir da entrega do desafio técnico, emitir um parecer fundamentado sobre o nível da pessoa candidata (Júnior, Pleno ou Sênior) e decidir com mais segurança sobre a contratação.

---

## Sumário

1. [Como usar este guia](#1-como-usar-este-guia)
2. [Mapa de senioridade](#2-mapa-de-senioridade)
3. [Rubrica de Backend (NestJS + Prisma)](#3-rubrica-de-backend-nestjs--prisma)
4. [Rubrica de Frontend (Next.js + TanStack Query + Tailwind)](#4-rubrica-de-frontend-nextjs--tanstack-query--tailwind)
5. [Rubrica de Testes](#5-rubrica-de-testes)
6. [Rubrica de Git e Histórico de Commits](#6-rubrica-de-git-e-histórico-de-commits)
7. [Rubrica de README e Documentação](#7-rubrica-de-readme-e-documentação)
8. [Pontos positivos e negativos por nível](#8-pontos-positivos-e-negativos-por-nível)
9. [Detecção de plágio e uso de IA](#9-detecção-de-plágio-e-uso-de-ia)
10. [Perguntas para a entrevista final](#10-perguntas-para-a-entrevista-final)
11. [Scorecard — planilha de pontuação](#11-scorecard--planilha-de-pontuação)

---

## 1. Como usar este guia

1. Leia o repositório entregue pelo candidato **na ordem cronológica dos commits** antes de abrir qualquer arquivo de código. O histórico de Git conta uma história — não pule para o estado final.
2. Aplique cada rubrica das seções 3–7, anotando pontos observados e ausências.
3. Use a seção 9 para verificar sinais de plágio ou geração automática por IA.
4. Combine as observações no scorecard (seção 11) para obter uma classificação de nível.
5. Selecione perguntas da seção 10 para a entrevista de validação humana. O objetivo é confirmar ou refutar a hipótese de nível levantada pelo código.

> **Princípio fundamental:** nenhum item isolado define o nível. O padrão geral e a consistência entre as dimensões (backend, frontend, testes, git, documentação) é o que importa.

---

## 2. Mapa de senioridade

A tabela abaixo resume o perfil esperado por nível. Use-a como âncora; os critérios detalhados estão nas seções seguintes.

| Dimensão | Júnior | Pleno | Sênior |
|---|---|---|---|
| **Domínio do problema** | Entrega o mínimo funcional, às vezes com erros de modelagem | Modela corretamente as entidades e os status | Modela com clareza, prevê extensibilidade, explica trade-offs |
| **Organização de código** | Tudo em um lugar (controller faz tudo) | Separação básica em camadas | Separação rigorosa, inversão de dependência, portas e adaptadores |
| **Qualidade de TypeScript** | `any` frequente, tipos básicos ou ausentes | Tipos corretos na maioria dos pontos, generics simples | Tipos rigorosos, sem `any`, utility types quando necessário |
| **Testes** | Ausentes ou um único teste de sanidade | Testes unitários da lógica de negócio | Pirâmide: unitários, integração e (se houver) e2e |
| **Frontend** | Estado local com `useState`, sem cache | TanStack Query correto, invalidação manual | TanStack Query com atualização otimista, tratamento de erros |
| **Git** | Poucos commits, mensagens genéricas | Commits atômicos razoáveis | Commits atômicos semânticos que contam a história do desenvolvimento |
| **Documentação** | README mínimo ou copiado do desafio | README funcional com instruções claras | README que explica decisões, trade-offs e uso consciente de IA |
| **Tratamento de erros** | Sem tratamento ou `console.log` | `try/catch` e respostas HTTP coerentes | Exceções de domínio, filtros globais, erros tipados |

---

## 3. Rubrica de Backend (NestJS + Prisma)

### 3.1 Estrutura de pastas e organização em camadas

**O que observar:**

```
# Estrutura mínima esperada (Pleno)
src/
  tasks/
    tasks.controller.ts
    tasks.service.ts
    tasks.module.ts
    dto/
      create-task.dto.ts
      update-task-status.dto.ts

# Estrutura diferenciada (Sênior)
src/
  tasks/
    domain/
      task.entity.ts
      task-status.enum.ts
      task.repository.ts        ← interface (porta)
    application/
      use-cases/
        create-task.use-case.ts
        update-task-status.use-case.ts
    infrastructure/
      persistence/
        prisma-task.repository.ts  ← implementação (adaptador)
    presentation/
      tasks.controller.ts
      dto/
```

| Nível | Indicadores |
|---|---|
| **Júnior** | Todo o código dentro do `AppModule` ou no controller; service mistura regra de negócio com chamada ao Prisma diretamente sem nenhuma abstração |
| **Pleno** | Pastas por funcionalidade (`tasks/`, `users/`), DTOs separados, service chamado pelo controller, Prisma usado dentro do service |
| **Sênior** | Inversão de dependência real (repository como interface injetada), camada de domínio sem dependência de framework, casos de uso isolados |

---

### 3.2 Modelagem do schema Prisma

**O que observar no `schema.prisma`:**

```prisma
# Mínimo esperado (Pleno)
model Task {
  id          String     @id @default(cuid())
  title       String
  description String?
  status      TaskStatus @default(TODO)
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
}

enum TaskStatus {
  TODO
  IN_PROGRESS
  DONE
}
```

**Pontos de atenção:**

- Uso de `String @id @default(uuid())` ou `cuid()` ao invés de `Int @id @autoincrement()` — indica consciência sobre sistemas distribuídos.
- Campos `createdAt` e `updatedAt` bem configurados.
- Enum de status no banco versus string livre — enum é a opção mais segura.
- Índices (`@@index`) em campos usados em filtros ou ordenações.

| Nível | Indicadores |
|---|---|
| **Júnior** | `id Int @id @autoincrement()`, status como `String` livre, sem `updatedAt`, sem pensar em extensibilidade |
| **Pleno** | UUID ou CUID, enum de status, `createdAt`/`updatedAt`, campos obrigatórios vs. opcionais corretos |
| **Sênior** | Schema pensado para o futuro (ex.: `priority`, `dueDate` como campos opcionais já previstos ou comentados), índices declarados, soft-delete com `deletedAt` ou justificativa de por que não fez |

---

### 3.3 Validação de entrada

**O que observar:**

- Uso de `class-validator` e `class-transformer` nos DTOs.
- `ValidationPipe` global no `main.ts` com `whitelist: true` e `forbidNonWhitelisted: true`.
- Mensagens de erro customizadas nos decorators.

```typescript
// Júnior — sem validação
@Post()
create(@Body() body: any) { ... }

// Pleno — validação correta
@IsString()
@IsNotEmpty()
@MaxLength(200)
title: string;

// Sênior — mensagens úteis, tipagem rigorosa
@IsString({ message: 'O título deve ser uma string' })
@IsNotEmpty({ message: 'O título é obrigatório' })
@MaxLength(200, { message: 'O título não pode ter mais de 200 caracteres' })
title: string;
```

---

### 3.4 Tratamento de erros

**O que observar:**

- Respostas HTTP coerentes: 404 quando não encontrado, 400 para dados inválidos, 409 para conflito, 500 para erros não tratados.
- Uso de `HttpException` ou exceções do NestJS (`NotFoundException`, `BadRequestException`).
- Filtro global de exceções customizado.

| Nível | Indicadores |
|---|---|
| **Júnior** | Nenhum tratamento; erro 500 para tudo; `try/catch` que apenas faz `console.log` |
| **Pleno** | `NotFoundException` quando tarefa não existe, `BadRequestException` para dados inválidos |
| **Sênior** | Exceções de domínio próprias (`TaskNotFoundException extends NotFoundException`), filtro global que formata a resposta de erro de forma consistente, logging estruturado |

---

### 3.5 Endpoints e semântica REST

**O que verificar:**

| Endpoint esperado | Método | Significado |
|---|---|---|
| `POST /tasks` | Criar tarefa | |
| `GET /tasks` | Listar todas as tarefas | |
| `PATCH /tasks/:id/status` | Atualizar status | Sênior prefere endpoint semântico ao `PATCH /tasks/:id` genérico |
| `GET /tasks/:id` | Buscar uma tarefa | Opcional mas esperado |
| `DELETE /tasks/:id` | Remover tarefa | Opcional |

**Armadilhas comuns:**
- Usar `PUT` onde deveria ser `PATCH` (atualização parcial).
- Endpoints que retornam 200 para operações que criaram algo (deveria ser 201).
- Body de resposta inconsistente entre endpoints.

---

### 3.6 Configuração e variáveis de ambiente

**O que observar:**

- `DATABASE_URL` não hardcoded no código — deve vir de `.env`.
- `.env.example` ou `.env.sample` presente no repositório.
- `.env` no `.gitignore`.
- Uso do `@nestjs/config` com schema de validação (`Joi` ou `zod`).

| Nível | Indicadores |
|---|---|
| **Júnior** | String de conexão hardcoded no código ou commitada no repositório |
| **Pleno** | `.env` no `.gitignore`, `.env.example` presente, variáveis lidas via `process.env` |
| **Sênior** | `ConfigModule` com validação de schema ao iniciar; aplicação falha na inicialização se variável obrigatória estiver ausente |

---

## 4. Rubrica de Frontend (Next.js + TanStack Query + Tailwind)

### 4.1 Uso do TanStack Query

Este é um dos pontos mais reveladores da entrega. É fácil instalar o TanStack Query; é difícil usá-lo corretamente.

**O que verificar:**

```typescript
// Júnior — TanStack Query instalado mas usado como useEffect
const [tasks, setTasks] = useState([]);
useEffect(() => {
  fetch('/api/tasks').then(r => r.json()).then(setTasks);
}, []);

// Pleno — uso correto básico
const { data: tasks, isLoading } = useQuery({
  queryKey: ['tasks'],
  queryFn: fetchTasks,
});
const mutation = useMutation({
  mutationFn: updateTaskStatus,
  onSuccess: () => queryClient.invalidateQueries({ queryKey: ['tasks'] }),
});

// Sênior — atualização otimista
const mutation = useMutation({
  mutationFn: updateTaskStatus,
  onMutate: async ({ taskId, newStatus }) => {
    await queryClient.cancelQueries({ queryKey: ['tasks'] });
    const previous = queryClient.getQueryData(['tasks']);
    queryClient.setQueryData(['tasks'], (old) =>
      old.map(t => t.id === taskId ? { ...t, status: newStatus } : t)
    );
    return { previous };
  },
  onError: (_, __, context) => {
    queryClient.setQueryData(['tasks'], context.previous);
  },
  onSettled: () => queryClient.invalidateQueries({ queryKey: ['tasks'] }),
});
```

| Nível | Indicadores |
|---|---|
| **Júnior** | `useEffect` + `fetch` manual; TanStack Query instalado mas nunca usado ou usado apenas para busca sem mutações |
| **Pleno** | `useQuery` e `useMutation` corretos; `invalidateQueries` após mutação; tratamento de `isLoading` e `isError` |
| **Sênior** | Atualização otimista; `queryKey` bem estruturado (ex.: `['tasks', { status: 'TODO' }]`); custom hooks que encapsulam as queries; separação entre a camada de API e os hooks |

---

### 4.2 Kanban e drag-and-drop

**O que observar:**

- Drag-and-drop implementado com biblioteca (`@hello-pangea/dnd`, `dnd-kit`) ou via HTML5 nativo.
- Feedback visual durante o arrasto.
- Estado do Kanban refletido no backend imediatamente após o drop (não apenas localmente).
- Tratamento de erro se a chamada ao backend falhar após o drop.

**Sinal de nível sênior:** drag-and-drop com rollback visual quando a chamada à API falha (atualização otimista com reversão).

---

### 4.3 Estrutura de componentes

**O que verificar:**

- Separação entre componentes de apresentação e containers/hooks.
- Componentes pequenos e coesos (um componente não faz tudo).
- Props com tipagem correta (sem `any`).
- Reutilização sensata (sem over-engineering: não criar abstração para algo usado uma vez).

| Nível | Indicadores |
|---|---|
| **Júnior** | Um único `page.tsx` com 400+ linhas; estado global no componente raiz; sem componentização significativa |
| **Pleno** | Componentes separados por responsabilidade (`TaskCard`, `KanbanColumn`, `TaskForm`); tipos corretos |
| **Sênior** | Custom hooks para lógica de negócio (`useTaskBoard`, `useCreateTask`); camada de serviço para chamadas à API; componentes que não conhecem a URL da API |

---

### 4.4 Fidelidade ao Figma e qualidade do CSS com Tailwind

**O que verificar:**

- Layout geral próximo ao design de referência (colunas Kanban, formulário de criação).
- Uso de Tailwind utilitário — sem CSS customizado desnecessário.
- Responsividade (pelo menos não quebrar em telas menores).
- Estado de loading com skeleton ou spinner.
- Estado vazio tratado (coluna vazia com mensagem).

**Armadilha comum:** candidatos que aplicam classes Tailwind aleatórias sem entender o sistema de espaçamento/cores — o resultado parece funcional mas visualmente inconsistente.

---

### 4.5 Next.js como SPA

O desafio exige Next.js 15+ rodando como **SPA**. Isso é um requisito explícito e revelador.

**O que verificar:**

- Configuração de `output: 'export'` no `next.config.ts` **ou** uso exclusivo de componentes client-side com `"use client"`.
- Ausência de Server Components para lógica de dados (dado que o backend já é o NestJS).
- Não usar `getServerSideProps` ou `server actions` para busca de dados — isso quebraria o requisito de SPA.

**Por que isso importa:** um candidato que mistura Server Components com TanStack Query de forma incorreta mostra que não leu o requisito ou não entende a diferença entre SSR e SPA.

---

## 5. Rubrica de Testes

### 5.1 Presença e estrutura

**Pergunta principal:** existe algum teste?

Se não existe nenhum teste, o candidato é, no máximo, **júnior** — independentemente da qualidade do restante do código.

| Nível | O que esperar |
|---|---|
| **Júnior** | Sem testes ou um único `it('should be defined')` copiado do scaffold do NestJS |
| **Pleno** | Testes unitários da camada de serviço com mocks do repositório; cobertura dos casos principais (criar tarefa, atualizar status, tarefa não encontrada) |
| **Sênior** | Pirâmide: unitários (domínio e casos de uso), integração (API com banco de dados real ou em memória), e2e opcional; testes que documentam o comportamento esperado |

---

### 5.2 Qualidade dos testes unitários

**O que observar:**

```typescript
// Teste fraco — testa implementação, não comportamento
it('deve chamar o repositório', async () => {
  await service.create(dto);
  expect(repository.create).toHaveBeenCalled();
});

// Teste forte — testa o comportamento esperado
it('deve lançar NotFoundException ao tentar atualizar status de tarefa inexistente', async () => {
  repository.findById.mockResolvedValue(null);
  await expect(service.updateStatus('id-falso', 'DONE')).rejects.toThrow(NotFoundException);
});
```

**Armadilhas comuns:**
- Testes que apenas verificam se funções foram chamadas (testes de implementação).
- Mocks que sempre retornam sucesso — nenhum caminho de erro testado.
- Nomes de teste genéricos (`it('funciona')`, `it('teste 1')`).

---

### 5.3 Testes de integração

**O que observar:**

- Uso de banco de dados real (ex.: PostgreSQL em container no CI) ou banco em memória (SQLite com Prisma).
- `supertest` para chamar endpoints HTTP reais.
- Seed de dados antes dos testes e limpeza depois.
- Teste da resposta HTTP completa (status code, body, headers).

---

## 6. Rubrica de Git e Histórico de Commits

### 6.1 Estrutura do histórico

O histórico de commits é uma das formas mais honestas de avaliar como alguém trabalha — especialmente porque candidatos tendem a preparar melhor o código do que o processo.

**O que verificar:**

```bash
git log --oneline
```

| Nível | Padrão observado |
|---|---|
| **Júnior** | 1 a 3 commits (`initial commit`, `done`, `fix`); ou o oposto: centenas de micro-commits sem semântica |
| **Pleno** | Commits por funcionalidade ou camada, mensagens descritivas em português ou inglês consistente |
| **Sênior** | Commits atômicos seguindo Conventional Commits (`feat:`, `fix:`, `test:`, `refactor:`, `chore:`); história que conta o raciocínio do desenvolvimento |

---

### 6.2 Conventional Commits (diferencial)

Candidatos que usam [Conventional Commits](https://www.conventionalcommits.org/) demonstram maturidade de processo:

```
feat(tasks): adiciona endpoint PATCH /tasks/:id/status
test(tasks): adiciona testes unitários para UpdateTaskStatusUseCase
fix(kanban): corrige reordenação visual após drag-and-drop
refactor(tasks): extrai interface TaskRepository do serviço
```

**Atenção:** commits com mensagens perfeitamente formatadas mas sem coerência temporal (todos no mesmo segundo ou hora) são um sinal de que o histórico foi reescrito ou gerado — veja a seção 9.

---

### 6.3 Diffs reveladores

Olhe os diffs individuais (`git show <hash>`). Procure por:

- **Commits grandes demais:** um único commit que adiciona backend + frontend + testes + docker ao mesmo tempo é sinal de que o candidato trabalhou tudo junto e fez um único commit no final.
- **Commits de "limpeza" antes da entrega:** commits como `refactor: clean up code` ou `chore: remove console.logs` logo antes da entrega podem indicar que o candidato poliu o trabalho só no final.
- **Reversões frequentes:** diffs que adicionam algo e depois removem na sequência mostram processo real de exploração e correção.

---

## 7. Rubrica de README e Documentação

### 7.1 O README como espelho do pensamento

O README de entrega é o lugar onde o candidato explica suas decisões. A qualidade do texto revela tanto quanto o código.

**O que verificar:**

- **Como rodar:** variáveis de ambiente, comandos para banco, migrations, backend e frontend — tudo presente e funcional?
- **Decisões técnicas:** o candidato explica *por que* escolheu determinada biblioteca ou abordagem? Ou apenas lista o que usou?
- **Trade-offs:** menciona o que *não* fez e por quê? Isso é sinal de maturidade.
- **Uso de IA:** descreve qual ferramenta, para quê usou e traz exemplo de prompt? (Este é um item explícito do desafio.)

| Nível | Padrão observado |
|---|---|
| **Júnior** | README copiado parcialmente do desafio; instruções genéricas que não funcionam ao seguir; sem explicação de decisões |
| **Pleno** | Instruções funcionais; menção às bibliotecas usadas; breve explicação das escolhas |
| **Sênior** | Explica o *raciocínio* por trás das decisões; menciona o que considerou mas descartou e por quê; uso de IA descrito de forma honesta e reflexiva |

---

### 7.2 Instruções de execução — teste prático

**Faça isso:** siga as instruções do README em um ambiente limpo (ou container) e veja se a aplicação sobe. Se não subir, isso é um bug de entrega — independentemente da qualidade do código.

**Pontos de falha comuns:**
- Variáveis de ambiente faltando no `.env.example`.
- Versão do Node não especificada.
- Falta do comando `npx prisma migrate dev` ou `npx prisma generate` nas instruções.
- Porta hardcoded que conflita com algo no ambiente do avaliador.

---

## 8. Pontos positivos e negativos por nível

### Júnior

**Pontos positivos (sinais de potencial):**
- Entregou algo funcional, mesmo que simples.
- README honesto sobre limitações.
- Fez perguntas relevantes ao longo do prazo (se houver troca de emails).
- Código legível, mesmo sem boas práticas avançadas.
- Mostrou curiosidade ao adicionar algo além do mínimo, mesmo que imperfeito.

**Pontos negativos (gaps esperados):**
- Ausência de testes — bloqueia confiança em evolução do código.
- Sem separação de camadas — dificulta manutenção.
- Sem tratamento de erros — aplicação quebra em cenários básicos.
- Sem validação de entrada — segurança e estabilidade comprometidas.
- Uso incorreto ou ausente do TanStack Query.
- Commits sem semântica.

---

### Pleno

**Pontos positivos:**
- Estrutura de código organizada e previsível.
- Testes unitários cobrindo os casos principais.
- TanStack Query usado corretamente.
- README funcional com decisões explicadas.
- Commits atômicos com mensagens descritivas.
- Tratamento de erros adequado.

**Pontos negativos (gaps que diferenciam do Sênior):**
- Sem atualização otimista no Kanban.
- Repositório acoplado ao Prisma diretamente (sem interface/abstração).
- Testes apenas unitários, sem integração.
- Sem validação de variáveis de ambiente.
- README lista o que fez, mas não explica o *porquê*.

---

### Sênior

**Pontos positivos:**
- Arquitetura permite trocar Prisma por outro ORM sem tocar na lógica de negócio.
- Atualização otimista com rollback visual no Kanban.
- Pirâmide de testes com cobertura real.
- README que conta uma história de decisões conscientes.
- Commits que documentam o processo de desenvolvimento.
- Exceções de domínio com hierarquia clara.
- Configuração de ambiente validada na inicialização.
- Uso de IA documentado de forma reflexiva, mostrando entendimento do que foi gerado.

**Pontos negativos (armadilhas mesmo em Sênior):**
- Overengineering: arquitetura hexagonal para uma tarefa simples sem justificativa.
- Testes lentos demais por excesso de integração onde unitários bastavam.
- README longo demais — sinal de que não sabe priorizar o que importa para o leitor.
- Abstrações prematuras que tornam o código mais difícil de ler.

---

## 9. Detecção de plágio e uso de IA

Esta seção lista sinais objetivos que indicam que o código foi gerado por IA (ChatGPT, Claude, Copilot, Cursor) **sem revisão crítica**, copiado de projetos externos ou que o candidato não tem domínio real sobre o que entregou.

> **Importante:** usar IA não é desonesto — o desafio inclusive incentiva e pede documentação do uso. O problema é quando a IA escreve tudo e o candidato não sabe explicar nada do que está no código.

---

### 9.1 Sinais de geração por IA sem revisão

**No código:**

| Sinal | Por quê é suspeito |
|---|---|
| Comentários que explicam o óbvio em todo arquivo | IA por padrão adiciona comentários explicativos que desenvolvedores experientes removem |
| Docstrings longas e perfeitas em todos os métodos | Padrão de output de LLMs; desenvolvedores reais comentam apenas o não-óbvio |
| Nomes excessivamente genéricos: `handleData`, `processItem`, `manageState` | LLMs tendem a nomear de forma segura e genérica |
| Código perfeitamente formatado em 100% dos arquivos sem nenhuma inconsistência | Nenhum desenvolvedor real é tão consistente; pequenas variações são normais |
| Imports organizados alfabeticamente em todos os arquivos | Comportamento típico de IA; desenvolvedores humanos raramente fazem isso |
| Zero `TODO` ou `FIXME` no código | Código real tem anotações de trabalho em progresso |
| Tratamento de erro para cenários impossíveis no contexto | IA adiciona `try/catch` "por precaução" mesmo onde o erro nunca pode ocorrer |
| Abstração excessiva para um problema simples | LLMs tendem a gerar código "enterprise-ready" para problemas simples |
| Código em inglês mas README em português literário perfeito (ou vice-versa) | Inconsistência de registro que indica fontes diferentes |
| Todos os casos de uso implementados com exatamente o mesmo padrão boilerplate | IA copia o próprio padrão com alta fidelidade |

**No histórico de Git:**

| Sinal | Por quê é suspeito |
|---|---|
| Commits com timestamps muito próximos (ex.: 10 commits em 5 minutos) | Sugestão de que foram gerados e commitados em lote |
| Commits com mensagens perfeitamente formatadas mas sem variação de estilo | Mensagens geradas por IA são uniformes demais |
| Um único commit inicial gigante com toda a implementação | Sugestão de que copiou/gerou tudo de uma vez |
| Histórico reescrito (`--force`) antes da entrega | Indica que o histórico foi "limpo" para esconder o processo real |
| Commits criados todos no mesmo horário do dia, todos os dias | Sugere trabalho em sessões únicas de geração em lote |

**No README:**

| Sinal | Por quê é suspeito |
|---|---|
| Linguagem de marketing ("solução robusta e escalável", "código limpo e bem estruturado") | Padrão de LLM sem edição humana |
| Seções genéricas que não se referem especificamente ao projeto | README copiado de template |
| Descrição de decisões técnicas que não aparecem no código | IA pode descrever o que "deveria" ter sido feito, não o que foi |
| Uso de IA descrito em termos vagos sem exemplo real de prompt | O desafio pede um exemplo de prompt; ausência ou vagueza é sinal |

---

### 9.2 Sinais de plágio de repositório externo

**O que verificar:**

1. **Pesquisar trechos únicos no Google/GitHub:** copie um bloco de código característico (ex.: um service method ou um custom hook) e pesquise entre aspas. Se aparecer em algum repositório público, é cópia.

2. **Verificar datas de criação vs. entrega:** se o repositório foi criado horas antes da entrega com um único commit gigante, isso é suspeito — especialmente se o código estiver completo demais para o prazo.

3. **Inconsistência de estilo entre arquivos:** arquivos escritos com estilos completamente diferentes (um usa `function`, outro usa arrow functions; um usa `interface`, outro usa `type`) podem indicar colagem de múltiplas fontes.

4. **Dependências que não aparecem no código:** se o `package.json` lista `lodash` mas nenhum arquivo o importa, foi copiado de outro projeto.

5. **Configurações específicas que não fazem sentido no contexto:** ex.: configurações de banco de dados para PostgreSQL e MySQL ao mesmo tempo, ou plugins de ESLint para Vue em um projeto Next.js.

---

### 9.3 Como distinguir uso honesto de IA vs. entrega desonesta

| Uso honesto e esperado | Uso problemático |
|---|---|
| Candidato lista a IA usada e explica como | Candidato nega uso de IA mas o código tem todos os sinais |
| Traz exemplo real de prompt e descreve o que mudou depois | README descreve uso de IA mas não há exemplo de prompt |
| Código gerado foi revisado, adaptado e testado | Código gerado está literalmente intacto, com comentários de IA |
| Candidato consegue explicar qualquer linha na entrevista | Candidato hesita ao explicar partes específicas do próprio código |
| Uso de IA para acelerar boilerplate, não para pensar o design | IA substituiu completamente o raciocínio arquitetural |

**A pergunta definitiva para a entrevista:** peça ao candidato para explicar uma decisão arquitetural específica e por que não fez de outra forma. Um desenvolvedor que entende o código consegue articular as trocas.

---

## 10. Perguntas para a entrevista final

As perguntas abaixo são organizadas por objetivo. Selecione entre 5 e 8 por candidato, priorizando as marcadas com (★) para candidatos cujo código levantou dúvidas.

### 10.1 Verificação de autoria e entendimento do código

Estas perguntas confirmam se o candidato entende o que entregou.

1. **(★) "Me mostra o fluxo completo de quando um usuário arrasta uma tarefa de 'A fazer' para 'Em andamento'. O que acontece no frontend e no backend, nessa ordem?"**
   > Candidatos que não escreveram o código hesitam em conectar as peças. Candidatos que escreveram respondem fluentemente, mesmo que o código não seja perfeito.

2. **(★) "Por que você escolheu usar [biblioteca X] ao invés de [alternativa Y]? Quais trade-offs pesaram nessa decisão?"**
   > Substitua X e Y por algo do código deles. Se não houver justificativa além de "é mais popular", é sinal de que a escolha foi automática.

3. "Se você fosse refatorar uma coisa no seu código depois dessa conversa, o que seria e por quê?"
   > Mede autocrítica e consciência de débito técnico.

4. **(★) "Esse trecho aqui [aponte um trecho específico] — você pode me explicar por que fez dessa forma e não de outra?"**
   > Escolha algo que pareceu gerado por IA. Um autor real tem uma resposta; quem colou não tem.

---

### 10.2 Profundidade técnica — Backend

5. "Sua arquitetura atual suporta facilmente adicionar um novo tipo de tarefa com campos diferentes. Como você faria isso sem quebrar o que já existe?"
   > Avalia extensibilidade e pensamento em domínio.

6. "O que acontece na sua aplicação se o banco de dados cair por 30 segundos? Como ela se comporta e o que o usuário vê?"
   > Avalia consciência de resiliência e tratamento de erros.

7. "Se amanhã precisássemos adicionar autenticação com JWT, onde no seu código isso se encaixaria? O que precisaria mudar?"
   > Avalia entendimento de separação de responsabilidades e extensibilidade.

8. "Seus testes usam o banco real ou mocks? Qual a vantagem da sua escolha? Em que situação você faria diferente?"
   > Avalia entendimento real de trade-offs de testes.

9. "Como você garantiria que dois usuários atualizando o status da mesma tarefa ao mesmo tempo não causem inconsistência?"
   > Avalia consciência de concorrência e transações — diferencial sênior.

---

### 10.3 Profundidade técnica — Frontend

10. "Me explica o que acontece no TanStack Query quando você chama `invalidateQueries`. O que é re-buscado, quando e por quê?"
    > Avalia entendimento real vs. uso mecânico da biblioteca.

11. "Se a chamada à API falhar depois que o usuário arrasta uma tarefa, o que acontece visualmente na tela? Isso é o comportamento ideal?"
    > Avalia se implementou atualização otimista e se pensa na experiência do usuário.

12. "O Next.js 15 tem Server Components. Por que você usou ou não usou Server Components para buscar as tarefas?"
    > Avalia entendimento do requisito de SPA e das implicações de cada abordagem.

13. "Me mostra como os `queryKey`s estão estruturados. Se você tivesse filtros por status, como mudaria a estrutura?"
    > Avalia entendimento do sistema de cache do TanStack Query.

---

### 10.4 Processo e colaboração

14. "Me fala sobre uma decisão técnica que você tomou e depois se arrependeu durante o desafio. O que você teria feito diferente?"
    > Avalia honestidade, capacidade de aprendizado e processo real de desenvolvimento.

15. "Se esse fosse um projeto real com mais de um desenvolvedor, o que você faria diferente no processo (não no código)?"
    > Avalia maturidade de trabalho em equipe — CI/CD, PR reviews, contratos de API, etc.

16. **(★) "Você mencionou que usou [ferramenta de IA]. Me mostra um exemplo concreto: qual era o problema, o que você pediu e o que mudou no output antes de commitar?"**
    > Confirma se o uso foi consciente e reflexivo, não apenas cópia cega.

17. "Dado o prazo de 7 dias, o que você priorizou e o que ficou de fora? Por quê essa ordem de prioridade?"
    > Avalia capacidade de priorização — uma das habilidades mais críticas em qualquer nível.

---

### 10.5 Perguntas de desempate (Pleno vs. Sênior)

Use quando o candidato está na fronteira entre os dois níveis.

18. "O que é inversão de dependência e como ela aparece (ou não) no seu código?"
    > Sênior explica e aponta onde aplicou ou onde saberia aplicar.

19. "Se eu te pedir para escrever um teste de integração para o endpoint `PATCH /tasks/:id/status`, como você estruturaria isso? Quais cenários cobriria?"
    > Avalia pensamento em testes além do unitário.

20. "Como você garantiria que a API do backend e o cliente do frontend nunca fiquem dessincronizados sobre os formatos de dados?"
    > Avalia conhecimento de contratos de API (OpenAPI, Zod compartilhado, etc.) — diferencial sênior.

---

## 11. Scorecard — planilha de pontuação

Use esta tabela para consolidar sua avaliação. Atribua de 1 a 3 em cada dimensão:
- **1 = Júnior** (entregou o básico com gaps significativos)
- **2 = Pleno** (entregou bem, com pontos de melhoria)
- **3 = Sênior** (acima do esperado para o nível)

| Dimensão | Peso | Nota (1–3) | Ponderado |
|---|---|---|---|
| Organização e arquitetura de backend | 20% | | |
| Modelagem de dados (Prisma) | 10% | | |
| Validação e tratamento de erros | 10% | | |
| Uso do TanStack Query e frontend | 15% | | |
| Fidelidade ao design (Figma) | 5% | | |
| Testes (presença + qualidade) | 20% | | |
| Histórico de Git | 10% | | |
| README e documentação | 10% | | |
| **Total ponderado** | **100%** | | |

**Interpretação do total ponderado:**

| Faixa | Nível sugerido |
|---|---|
| 1.0 – 1.5 | Júnior |
| 1.5 – 2.3 | Pleno |
| 2.3 – 3.0 | Sênior |

> **Lembrete:** o scorecard é uma ferramenta de apoio, não um veredito automático. A entrevista final (seção 10) pode tanto confirmar quanto contradizer a pontuação — e deve sempre ter o peso final na decisão.

---

### Decisão final

Após o scorecard e a entrevista, registre aqui:

```
Candidato(a): _______________________________________________
Data da avaliação: __________________________________________
Avaliador(a): _______________________________________________

Nível identificado no código:  [ ] Júnior  [ ] Pleno  [ ] Sênior
Nível confirmado na entrevista: [ ] Júnior  [ ] Pleno  [ ] Sênior

Sinais de uso de IA identificados: [ ] Não  [ ] Sim (honesto)  [ ] Sim (problemático)
Sinais de plágio identificados:    [ ] Não  [ ] Suspeito       [ ] Confirmado

Pontos fortes:
- 
- 
- 

Gaps principais:
- 
- 
- 

Recomendação: [ ] Contratar  [ ] Contratar com ressalvas  [ ] Não contratar

Justificativa:
_______________________________________________________________
_______________________________________________________________
```

---

*Documento elaborado para uso interno da Academia do Universitário. Revisão recomendada a cada ciclo de contratação.*

---

## 12. Template de resultado para o candidato

> Copie o bloco abaixo, preencha e envie ao candidato após a decisão. O template tem duas versões: uma para aprovação e uma para reprovação. Use apenas a versão adequada e delete a outra antes de enviar.

````markdown
<!-- ============================================================ -->
<!-- VERSÃO APROVAÇÃO — delete este bloco se for reprovação       -->
<!-- ============================================================ -->

# Feedback — Desafio Técnico Full Stack | Academia do Universitário

Olá, [Nome do candidato(a)]!

Avaliamos com atenção sua entrega do desafio técnico e gostaríamos de compartilhar nosso feedback detalhado.

## Resultado

**Nível identificado:** [Júnior / Pleno / Sênior]
**Decisão:** Aprovado(a) para a próxima etapa.

## O que se destacou positivamente

<!-- Liste de 2 a 4 pontos concretos e específicos do código/entrega -->
- [ex.: A separação entre casos de uso e repositório no backend mostrou maturidade arquitetural acima do esperado para o nível.]
- [ex.: O uso do TanStack Query com atualização otimista e rollback visual foi implementado corretamente e demonstra cuidado com a experiência do usuário.]
- [ex.: O histórico de commits contou a história do desenvolvimento de forma clara, com mensagens atômicas e semânticas.]

## Pontos de atenção e oportunidades de melhoria

<!-- Liste de 1 a 3 gaps observados — seja específico, não genérico -->
- [ex.: Os testes cobriram bem os casos de sucesso, mas os caminhos de erro (tarefa não encontrada, status inválido) ficaram sem cobertura.]
- [ex.: A validação das variáveis de ambiente na inicialização poderia ser mais rigorosa — uma variável faltando só causaria erro em runtime.]

## Próximos passos

Entraremos em contato em até [X dias úteis] para agendar [a próxima etapa / uma conversa com o time].

Parabéns pela entrega e até breve!

**[Nome do avaliador(a)]**
Academia do Universitário

---

<!-- ============================================================ -->
<!-- VERSÃO REPROVAÇÃO — delete este bloco se for aprovação       -->
<!-- ============================================================ -->

# Feedback — Desafio Técnico Full Stack | Academia do Universitário

Olá, [Nome do candidato(a)]!

Avaliamos com atenção sua entrega do desafio técnico e gostaríamos de compartilhar nosso feedback.

## Resultado

**Nível identificado:** [Júnior / Pleno / Sênior]
**Decisão:** Não seguiremos com o processo neste momento.

Essa decisão não é definitiva sobre o seu potencial — ela reflete o alinhamento entre o perfil atual e as necessidades específicas desta vaga.

## O que se destacou positivamente

<!-- Sempre mencione algo genuíno. Feedback vazio ou genérico é pior do que nenhum feedback. -->
- [ex.: A entrega foi funcional e o README estava bem organizado, com instruções claras para rodar o projeto.]
- [ex.: A estrutura de componentes no frontend mostrou bom entendimento de separação de responsabilidades.]

## Por que não seguimos adiante

<!-- Seja honesto e específico. Evite eufemismos como "perfil não adequado" sem explicação. -->
- [ex.: A vaga exige experiência comprovada com testes — tanto unitários quanto de integração — e a entrega não continha nenhum teste. Isso é um requisito não negociável para o time atual.]
- [ex.: A separação de camadas no backend ficou abaixo do esperado para o nível Pleno: a lógica de negócio estava misturada com as chamadas ao banco de dados no controller.]
- [ex.: O TanStack Query foi instalado mas as queries foram implementadas com `useEffect` + `fetch` manual, o que sugere que a ferramenta ainda não foi absorvida na prática.]

## O que sugerimos para continuar evoluindo

<!-- Opcional, mas muito valorizado pelos candidatos. Seja construtivo. -->
- [ex.: Praticar a pirâmide de testes em projetos pessoais — começar por testes unitários da camada de serviço com Jest é um bom ponto de entrada.]
- [ex.: Explorar o guia oficial do TanStack Query, especialmente as seções de mutações e atualização otimista.]
- [ex.: Construir um projeto pessoal com arquitetura em camadas explícita ajuda a solidificar a separação entre domínio, aplicação e infraestrutura.]

Agradecemos o tempo e o esforço dedicados ao desafio. Desejamos sucesso na sua jornada!

**[Nome do avaliador(a)]**
Academia do Universitário
````
