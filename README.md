# Plataforma de Gerenciamento de Tarefas

Este é um projeto de arquitetura de microsserviços que simula uma plataforma de gerenciamento de tarefas. Ele é composto por um Frontend em React, um Backend de Usuários em Go, um Backend de Tarefas em Python (FastAPI), um Microsserviço de Notificações em Node.js e um Banco de Dados PostgreSQL, todos orquestrados via Docker Compose e com um Nginx atuando como API Gateway.

## Visão Geral da Arquitetura

A aplicação é dividida em cinco microsserviços principais, um sistema de fila de mensagens e um banco de dados, comunicando-se em uma rede Docker interna:

-   **`frontend`**: Aplicação web em **React (Vite)**, servindo a interface do usuário.
-   **`api-gateway`**: Servidor **Nginx** que atua como ponto de entrada único para todas as requisições do frontend, roteando-as para os backends apropriados e gerenciando CORS.
-   **`users-service`**: Microsserviço de **Go (Golang)** responsável pela autenticação (registro e login) e gerenciamento de usuários.
-   **`tasks-service`**: Microsserviço de **Python (FastAPI)** responsável pelo CRUD (Criação, Leitura, Atualização, Exclusão) de tarefas.
-   **`notifications-service`**: Microsserviço de **Node.js** responsável pelo envio assíncrono de notificações (e-mail, etc.) em resposta a eventos do sistema (ex: tarefa concluída). Ele se comunicará via fila de mensagens.
-   **`rabbitmq`**: Um **broker de mensagens** (fila) que permitirá a comunicação assíncrona entre o `tasks-service` (que publicará eventos) e o `notifications-service` (que consumirá esses eventos).
-   **`db`**: Banco de dados **PostgreSQL** para persistência dos dados de usuários e tarefas.

<img width="1469" height="614" alt="image" src="https://github.com/user-attachments/assets/6a5dfdd3-cc09-4b8e-9b32-9932ecbff5e4" />

## Pré-requisitos

Antes de iniciar, certifique-se de ter os seguintes softwares instalados em seu ambiente:

-   **Git**: Para clonar o repositório.
-   **Docker Desktop**: Inclui Docker Engine, Docker CLI e Docker Compose. Essencial para rodar todos os serviços. Certifique-se de que a **integração com WSL 2 para Ubuntu** esteja habilitada nas configurações do Docker Desktop.
-   **WSL (Windows Subsystem for Linux)** com Ubuntu: O ambiente de desenvolvimento primário.
-   **Node.js e npm (no WSL)**: Necessário para o desenvolvimento do frontend (React) e do `notifications-service`. Versão LTS recomendada (ex: Node.js 20).
    -   Você pode instalar via `nvm` (Node Version Manager) no seu WSL.
-   **Go (no WSL)**: Necessário para o desenvolvimento do `users-service`. Versão 1.23 ou superior recomendada.
    -   Certifique-se de que o `go` está no seu `PATH` (ex: configurando `.bashrc` ou `.zshrc`).
-   **Python (no WSL) e pip/pipenv**: Necessário para o desenvolvimento do `tasks-service`. Versão 3.10 ou superior recomendada.
    -   Você pode usar `pyenv` para gerenciar as versões do Python.
-   **VS Code (no Windows)**: Com a extensão "WSL" instalada para desenvolvimento integrado.

## Configuração do Ambiente Local

Siga estes passos para configurar e rodar a aplicação em seu ambiente local:

1.  **Clone o Repositório:**
    Abra seu terminal WSL (Ubuntu) e clone este repositório:
    ```bash
    git clone [URL_DO_SEU_REPOSITORIO]
    cd plataforma-de-gerenciamento-de-tarefas # Ou o nome da pasta raiz do seu projeto
    ```

2.  **Preparar Microsserviços Individuais:**

    * **Frontend (`frontend` - React):**
        ```bash
        cd frontend
        npm install # Instala as dependências do Node.js
        # Crie ou verifique o arquivo .env:
        echo "VITE_API_BASE_URL=http://localhost" > .env # Se já existe, apenas verifique o conteúdo
        cd ..
        ```

    * **Microsserviço de Usuários (`users-service` - Go):**
        ```bash
        cd users-service
        go mod tidy # Baixa as dependências e gera go.sum
        cd ..
        ```

    * **Microsserviço de Tarefas (`tasks-service` - Python):**
        ```bash
        cd tasks-service
        pip install -r requirements.txt # Instala as dependências Python no ambiente WSL (opcional, para editor)
        cd ..
        ```
    * **Microsserviço de Notificações (`notifications-service` - Node.js):**
        ```bash
        cd notifications-service
        # Instalar dependências Node.js aqui quando o código for criado (ex: npm install)
        cd ..
        ```

3.  **Iniciar a Aplicação com Docker Compose:**
    Na **raiz do projeto** (onde está o `docker-compose.yml`), execute:
    ```bash
    docker compose down --volumes --remove-orphans # Para garantir um início limpo
    docker compose up -d --build # Constrói/reconstrói todas as imagens e inicia os serviços em background
    ```

4.  **Verificar o Status dos Serviços:**
    Certifique-se de que todos os contêineres estão rodando:
    ```bash
    docker compose ps
    ```
    Você deve ver `db`, `rabbitmq`, `users-service`, `tasks-service`, `notifications-service`, `api-gateway` e `frontend` com status `running` (ou `healthy`).

5.  **Acessar a Aplicação:**
    Abra seu navegador web e acesse:
    -   **Frontend:** `http://localhost:5173`
    -   **Documentação do Serviço de Usuários (Swagger UI):** `http://localhost:8001/docs` (acesso direto, ignorando Gateway)
    -   **Documentação do Serviço de Tarefas (Swagger UI):** `http://localhost:8002/docs` (acesso direto, ignorando Gateway)

## Como Usar a Aplicação

1.  **Registro de Usuário:**
    -   Acesse `http://localhost:5173` (ou a página de registro se já estiver configurada no frontend).
    -   Preencha o formulário para criar um novo usuário.
    -   **Alternativamente (via API Gateway):**
        ```bash
        curl --location 'http://localhost/users/register' \
        --header 'Content-Type: application/json' \
        --data '{
            "username": "seu_usuario",
            "password": "senha_segura_123",
            "email": "seu.email@example.com"
        }'
        ```

2.  **Login de Usuário:**
    -   Após o registro, faça login pelo frontend.
    -   **Alternativamente (via API Gateway):**
        ```bash
        curl --location 'http://localhost/users/login' \
        --header 'Content-Type: application/json' \
        --data '{
            "username": "seu_usuario",
            "password": "senha_segura_123"
        }'
        ```
        **Guarde o token JWT retornado**, ele será necessário para acessar os endpoints protegidos.

3.  **Gerenciar Tarefas (Exemplo no Postman/Curl - o frontend ainda será implementado):**
    -   **Listar Tarefas (GET):**
        ```bash
        curl --location 'http://localhost/tasks' \
        --header 'Authorization: Bearer SEU_TOKEN_JWT_AQUI'
        ```
    -   **Criar Tarefa (POST):**
        ```bash
        curl --location --request POST 'http://localhost/tasks' \
        --header 'Content-Type: application/json' \
        --header 'Authorization: Bearer SEU_TOKEN_JWT_AQUI' \
        --data '{
            "title": "Minha Primeira Tarefa",
            "description": "Exemplo de tarefa criada via API Gateway.",
            "status": "pending"
        }'
        ```
        (Substitua `SEU_TOKEN_JWT_AQUI` pelo token obtido no login).

## Parar a Aplicação

Para parar e remover todos os contêineres e redes criadas pelo Docker Compose:

```bash
cd /caminho/para/a/raiz/do/projeto
docker compose down
docker compose up -d --build

