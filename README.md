# sobreOAuth

## Sobre sobreOAuth

Estudo e compilação sobre o protocolo OAuth.

> ***Alguns termos serão mantidos em inglês.***

**O que é OAuth 2.0?**

É um protocolo de autorização (não autenticação) que permite que aplicações terceiras acessem recursos de um usuário em um serviço (como Google, GitHub, etc.) sem compartilhar suas credenciais.

> ***Exemplo:*** Um app de fotos que acessa suas imagens do Google Drive sem saber sua senha.

**Diferença entre OAuth 2.0 e OpenID Connect (OIDC)?**

- OAuth 2.0: Focado em autorização (acessar recursos).
- OIDC: Camada sobre OAuth 2.0 que adiciona autenticação (saber quem é o usuário via id_token JWT).

## Referências

- [(Udemy) The Nuts and Bolts of OAuth 2.0](https://www.udemy.com/course/oauth-2-simplified)
- [(Udemy) Advanced OAuth Security](https://www.udemy.com/course/advanced-oauth-security/)
- [The Little Book of OAuth 2.0 RFCs](https://oauth.net/books/The%20Little%20Book%20of%20OAuth%202.0%20RFCs.pdf)
- [`oauth.net`](https://oauth.net/)
  - [OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified/)
  - [OAuth 2.0 Servers](https://www.oauth.com/)
- [(Udemy) Master OAuth 2.0: A Practical Guide to API Security](https://www.udemy.com/course/master-oauth-2-api-security-practical-guide)
- [(Udemy) Web Authentication With Golang - Google's Go Language](https://www.udemy.com/course/oauth-authentication/)
- [RFC list](https://www.rfc-editor.org/search/rfc_search_detail.php?title=OAuth&page=All)
- OAuth 2.0 em português claro:
  - [Parte I](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-i-23c5618a4601)
  - [Parte II](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-ii-e8e0d8dba994)
  - [Parte III](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-iii-669a07807f9a)

Ferramentas para Testar:

- Postman: Suporte nativo a fluxos OAuth.
- oauth.tools: Site interativo para simular fluxos.
- Keycloak: Servidor de autorização open-source.

## Mapa das Especificações do OAuth 2.0

> `https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/`

The OAuth 2.0 Core Framework (RFC 6749) defines roles and a base level of functionality, but leaves a lot of implementation details unspecified. Since the publication of the RFC, the OAuth Working Group has published many additional specs built on top of this framework to fill in the missing pieces. Looking at the full list of specs the group is working on can be somewhat overwhelming. This chapter lays out how the various specs relate to each other.

## Terminologia

Regras:

- Resource owner: o usuário dono dos dados (ex.: sua conta do Google).
- User agent: o dispositivo
- Client: a aplicação que quer acessar os dados (ex.: um app terceiro).
  - Confidential clients (no servidor): tem as credenciais
  - Public clients (browser web ou aplicativos móveis): não tem as credenciais
- Resource server: servidor que armazena os dados protegidos (ex.: API do Google Drive).
- Authorization server: servidor que valida o usuário e emite tokens (ex.: login do Google).

Tokens:

- Authorization code: `client_secret` armazenado por um back-end seguro.
- Access token: "chave" temporária que o Client usa para acessar o Resource Server.
- Refresh token: token usado para obter novos Access Tokens sem pedir permissão novamente ao usuário.

Grant types (a.k.a. Grant flows / Authorization flows / Fluxos):

- **Authorization Code** (mais comum)
  - Client redireciona o usuário para o AS (ex.: tela de login do Google).
  - Usuário autoriza e o AS envia um código de autorização para o Client.
  - Client troca esse código por um Access Token (requisição back-end).
    - Vantagem: Seguro (token nunca exposto no front-end).
- **PKCE** (Authorization Code + segurança extra)
  - Similar ao anterior, mas com um code challenge para evitar ataques em apps móveis/SPA.
  - Obrigatório para clientes públicos (como apps JavaScript).
- **Client Credentials** (máquina-a-maquina, service-to-service)
  - Usado para comunicação entre serviços (ex.: microserviços).
  - O Client se autentica diretamente no AS com seu client_id e client_secret.
- **Device Code** (browseless ou input constraint devices)
  - dispositivos sem teclado/tela (ex.: Smart TVs, IoT) ou onde a entrada de credenciais é inviável.
- **Refresh Token**
  - Permite obter novos Access Tokens após a expiração sem interação do usuário (usuário ter que efetuar novo login).
- **Implicit Flow** (depreciado, legacy)
  - Antigo fluxo para SPAs, agora substituído pelo PKCE.
  - Obtinha o Access Token diretamente sem o Authorization Code.
- **Password Grant** (depreciado, legacy)
  - Pede ao usuário a senha.
  - Aplicações legadas ou de alta confiança (ex.: app oficial do próprio provedor de serviço).
  - Não recomendado para clientes públicos (vazamento de senhas).

Scopes e Permissões:

- Scopes definem quais permissões o Client pede (ex.: read:contacts, write:files).
- O usuário deve consentir explicitamente com esses scopes durante a autorização.
- Define o conjunto de informações do cliente que serão expostas pelo token.

## Front Channel vs. Back Channel

...

## Boas Práticas e Segurança

- Nunca armazene tokens no front-end (localStorage/sessionStorage).
- Use HTTPS em todas as etapas.
- Para SPAs/mobile, prefira PKCE.
- Valide sempre os scopes no Resource Server.
- Short-lived Access Tokens + Refresh Tokens são a melhor prática.

