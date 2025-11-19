# Prática 07 - Segurança - Spring Boot + JWT + Keycloak

Este projeto é uma implementação de uma API REST com Spring Boot protegida por JWT (JSON Web Tokens) e utilizando o Keycloak como servidor de autorização (Authorization Server), conforme o roteiro da Prática 07.

## 1. Requisitos

*   Java 17+
*   Maven
*   Keycloak (versão 21+ recomendada)
*   Postman ou `curl` para testes

## 2. Configuração do Keycloak

Para que a aplicação funcione, é necessário configurar o Keycloak.

### Passos:

1.  **Instalar e Iniciar o Keycloak:**
    Inicie o Keycloak localmente (geralmente acessível em `http://localhost:8080`).
2.  **Criar um Realm:**
    Crie um novo Realm, por exemplo, `demo`.
3.  **Criar um Client:**
    Crie um Client chamado `spring-api` com as seguintes configurações:
    *   **Access Type:** `confidential`
    *   **Standard Flow Enabled:** `true`
    *   **Direct Access Grants Enabled:** `true`
4.  **Criar Roles:**
    No Realm `demo`, crie as roles: `user` e `admin`.
5.  **Criar Usuários e Atribuir Roles:**
    Crie usuários e atribua a eles as roles criadas. Por exemplo:
    *   Usuário `joao`: role `user`
    *   Usuário `maria`: roles `user` e `admin`
6.  **Atualizar `application.yml`:**
    O arquivo `src/main/resources/application.yml` deve ter o `issuer-uri` configurado para o seu Realm.
    ```yaml
    spring:
      security:
        oauth2:
          resourceserver:
            jwt:
              issuer-uri: http://localhost:8080/realms/demo # Verifique se esta URL está correta
    ```

## 3. Como Rodar a Aplicação

1.  **Compilar e Empacotar:**
    ```bash
    mvn clean install
    ```
2.  **Executar:**
    ```bash
    mvn spring-boot:run
    ```
    A aplicação estará rodando em `http://localhost:8080` (ou outra porta configurada).

## 4. Testes e Exemplos de Requisições

Para testar os endpoints, você precisará de um **Token JWT** válido, obtido no Keycloak.

### 4.1. Obter o Token JWT

Você pode usar o **Direct Access Grants** do Keycloak para obter um token via `curl`.

**Exemplo (substitua com suas credenciais e client secret):**

```bash
curl -X POST "http://localhost:8080/realms/demo/protocol/openid-connect/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=password" \
-d "client_id=spring-api" \
-d "client_secret=<SEU_CLIENT_SECRET>" \
-d "username=<SEU_USUARIO>" \
-d "password=<SUA_SENHA>"
```

A resposta conterá o `access_token`.

### 4.2. Testar Endpoints

Use o token obtido no header `Authorization: Bearer <seu_token_jwt>`.

#### Endpoint Público (`/public`)

*   **Acesso:** Livre (não requer autenticação).
*   **Requisição:**
    ```bash
    curl http://localhost:8080/public
    ```
*   **Resposta Esperada:** `Acesso público`

#### Endpoint de Usuário (`/user`)

*   **Acesso:** Requer autenticação (qualquer usuário com token válido).
*   **Requisição:**
    ```bash
    curl -H "Authorization: Bearer <seu_token_jwt>" http://localhost:8080/user
    ```
*   **Resposta Esperada:** `Acesso autenticado`

#### Endpoint de Administrador (`/admin`)

*   **Acesso:** Requer autenticação e a role `admin`.
*   **Requisição:**
    ```bash
    curl -H "Authorization: Bearer <seu_token_jwt>" http://localhost:8080/admin
    ```
*   **Resposta Esperada (com role `admin`):** `Acesso restrito a admins`
*   **Resposta Esperada (sem role `admin`):** Erro 403 Forbidden
