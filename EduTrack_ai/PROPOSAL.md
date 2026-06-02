# PROPOSAL: EduTrack AI

Este documento serve como proposta e especificação técnica detalhada do projeto **EduTrack AI**. Ele contém todas as informações necessárias para que uma Inteligência Artificial ou equipe de desenvolvedores consiga construir o projeto do zero, separando em backend e frontend.

## 1. Visão Geral do Projeto
O **EduTrack AI** é uma plataforma de gestão acadêmica que permite aos alunos organizar suas disciplinas, controlar tarefas, visualizar estatísticas de desempenho através de um dashboard interativo, e gerar insights baseados em Inteligência Artificial sobre sua produtividade e organização.

## 2. Stack Tecnológico

### Backend
- **Linguagem:** Java 17
- **Framework:** Spring Boot 3.x
- **Dependências Principais:**
  - Spring Web
  - Spring Data JPA
  - Spring Security & OAuth2 Client
  - Spring Validation
  - Banco de Dados: MySQL (MySQL Connector/J)
  - Autenticação: JJWT (JSON Web Token)
  - Documentação da API: Springdoc OpenAPI (Swagger)
  - Inteligência Artificial: Google GenAI SDK (`com.google.genai:google-genai`)
  - Utilitários: Lombok, Jackson Kotlin Module

### Frontend
- **Linguagem/Framework:** JavaScript com React (via Vite)
- **Roteamento:** React Router DOM
- **Requisições HTTP:** Axios
- **Gráficos:** Recharts
- **Autenticação:** `@react-oauth/google` (Google OAuth)
- **Estilização:** CSS puro (Vanilla CSS) com variáveis para temas (Dark/Light mode).

---

## 3. Banco de Dados (Entidades e Relacionamentos)

O sistema utiliza um banco de dados relacional (MySQL) com as seguintes entidades:

### 3.1. Usuario (Tabela: `usuarios`)
- `id` (Long, PK, Auto Increment)
- `nome` (String, length=120, Not Null)
- `email` (String, length=180, Not Null, Unique)
- `senha` (String, length=255, Nullable - *nulo caso login seja social*)
- `fotoUrl` (String, LONGTEXT, Nullable)
- **Relacionamentos:**
  - `OneToMany` com `SocialLogin` (cascade ALL, orphanRemoval true)
  - `OneToMany` com `Disciplina` (cascade ALL, orphanRemoval true)

### 3.2. SocialLogin (Tabela: `social_logins`)
*Armazena os vínculos de provedores externos (ex: Google) com o usuário.*
- `id` (Long, PK, Auto Increment)
- `provider` (String, length=60, Not Null)
- `providerId` (String, length=255, Not Null)
- `usuario_id` (Long, FK, Not Null)
- **Constraints:** Unique Constraint combinada em (`provider`, `providerId`)

### 3.3. Disciplina (Tabela: `disciplinas`)
- `id` (Long, PK, Auto Increment)
- `nome` (String, length=120, Not Null)
- `professor` (String, length=120, Not Null)
- `cargaHoraria` (Integer, Not Null)
- `descricao` (String, TEXT, Not Null)
- `dataInicio` (LocalDate, Not Null)
- `dataFim` (LocalDate, Not Null)
- `usuario_id` (Long, FK, Not Null)
- **Relacionamentos:**
  - `ManyToOne` com `Usuario`
  - `OneToMany` com `Tarefa` (cascade ALL, orphanRemoval true)

### 3.4. Tarefa (Tabela: `tarefas`)
- `id` (Long, PK, Auto Increment)
- `titulo` (String, length=160, Not Null)
- `descricao` (String, TEXT, Not Null)
- `dataEntrega` (LocalDate, Not Null)
- `status` (Enum `StatusTarefa`, length=30, Not Null)
  - Valores do Enum: `PENDENTE`, `EM_ANDAMENTO`, `CONCLUIDA`
- `disciplina_id` (Long, FK, Not Null)
- **Relacionamentos:**
  - `ManyToOne` com `Disciplina`

---

## 4. Backend: API Endpoints

Abaixo estão os endpoints disponíveis no sistema. Todos os endpoints (exceto os de registro, login, e recuperação de senha) são protegidos via Token JWT no header `Authorization: Bearer <token>`.

### 4.1. Autenticação (`/auth`)
- `POST /auth/cadastro`: Cria um novo usuário tradicional (nome, email, senha).
- `POST /auth/login`: Autentica um usuário e retorna o token JWT.
- `POST /auth/login/oauth2/google`: Autentica/Cadastra o usuário via Google e retorna o token JWT.
- `POST /auth/esqueci-senha`: Solicita código de recuperação enviando email.
- `POST /auth/verificar-codigo`: Valida o código recebido no email.
- `POST /auth/reseta-senha`: Define a nova senha.

### 4.2. Usuário (`/users`)
- `GET /users/me`: Retorna os dados do usuário autenticado.
- `PUT /users/me`: Atualiza o perfil (nome, email, senha).
- `DELETE /users/me`: Exclui a conta do usuário (junto com dados atrelados).
- `POST /users/me/foto`: Atualiza a foto do perfil (fotoUrl).

### 4.3. Disciplinas (`/disciplinas`)
- `GET /disciplinas`: Lista as disciplinas do usuário de forma paginada (com filtro de `search` opcional).
- `POST /disciplinas`: Cria uma nova disciplina.
- `GET /disciplinas/resumo`: Retorna totalizador das disciplinas.
- `PUT /disciplinas/{id}`: Edita os dados da disciplina.
- `DELETE /disciplinas/{id}`: Deleta uma disciplina.

### 4.4. Tarefas (`/tarefas`)
- `GET /tarefas`: Lista tarefas paginadas (Filtros: `disciplinaId`, `status`, `atrasadas`).
- `GET /tarefas/estatisticas`: Retorna totalizadores por status e métricas.
- `POST /tarefas`: Cria uma nova tarefa.
- `PUT /tarefas/{id}`: Edita os dados da tarefa.
- `DELETE /tarefas/{id}`: Deleta uma tarefa.

### 4.5. Dashboard (`/dashboard`)
- `GET /dashboard/resumo`: KPIs principais (total disciplinas, tarefas, concluidas, pendentes).
- `GET /dashboard/graficos/disciplinas`: Dados para o gráfico de tarefas por disciplina.
- `GET /dashboard/graficos/tarefas-por-status`: Dados para o gráfico de pizza (Pendente, Andamento, Concluída).
- `GET /dashboard/disciplinas`: Lista sumarizada das disciplinas para o quadro.
- `GET /dashboard/tarefas-prioritarias`: Lista as tarefas mais urgentes baseada na `dataEntrega`.

### 4.6. Insights de Inteligência Artificial (`/insights`)
- `POST /insights/generate`: Consulta a IA (Google GenAI) enviando o contexto das disciplinas e tarefas do aluno, retornando dicas personalizadas de organização e estudos.

---

## 5. Frontend: Estrutura e Telas

O frontend foi desenvolvido com Vite + React. A estrutura principal é dividida em Rotas (Telas), Componentes globais e Serviços.

### 5.1. Telas (Pages/Rotas)
1. **TelaLanding**: Página inicial do sistema apresentando o EduTrack AI, features e call to action para login/cadastro.
2. **TelaLogin**: Formulário de login tradicional e botão para Login com Google.
3. **TelaCadastro**: Formulário para criar uma nova conta de usuário.
4. **TelaEsqueciSenha**: Fluxo para recuperação de senha (enviar email -> código -> nova senha).
5. **TelaDashboard** (Protegida): Visão geral do aluno. Exibe os gráficos (Recharts), resumo das métricas, lista de disciplinas e tarefas prioritárias.
6. **TelaDisciplinas** (Protegida): CRUD completo (Listagem com busca, Criação, Edição, Exclusão) de disciplinas.
7. **TelaTarefas** (Protegida): CRUD de tarefas. Listagem filtrável por status e disciplina, além da criação e deleção.
8. **TelaInsights** (Protegida): Interface que consome o endpoint do Google GenAI para mostrar dicas de estudo e recomendações com base no progresso das tarefas do aluno.
9. **TelaPerfil** (Protegida): Permite que o usuário visualize seus dados, altere nome/senha, altere foto de perfil e exclua a conta.

### 5.2. Componentes Reutilizáveis
- **NavbarPrincipal**: Barra de navegação do topo (Menu lateral para mobile, links, controle de tema e menu do usuário).
- **RotaProtegida**: Componente Wrapper (HOC) que bloqueia o acesso a rotas privadas caso não exista o Token JWT no localStorage.
- **ToggleTema**: Botão de switch para alternar dinamicamente entre tema Claro e Escuro (via manipulação de variáveis CSS no :root ou classe no body).
- **Toast**: Componente para exibir notificações temporárias de sucesso, erro ou alerta na tela.
- **ModalFotoPerfil**: Modal de interface para colar a URL ou fazer o upload da nova foto.
- **IconeVisibilidade**: Ícone customizado (olho/olho cortado) para exibir/ocultar senha nos inputs.

### 5.3. Serviços e Configurações
- **`api.js`**: Instância customizada do Axios configurada com a `baseURL` do backend (ex: `http://localhost:8080`) e um `Interceptor` para injetar o header `Authorization: Bearer token` de forma automática em todas as requisições protegidas.
- Estilização com **Vanilla CSS** modularizado, onde cada componente e tela tem seu próprio arquivo `.css` importado (ex: `TelaDashboard.css`), gerenciando um padrão global com o `index.css` e `App.css`.
