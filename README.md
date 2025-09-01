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

### Links

- [(Udemy) The Nuts and Bolts of OAuth 2.0](https://www.udemy.com/course/oauth-2-simplified)
- [(Udemy) Advanced OAuth Security](https://www.udemy.com/course/advanced-oauth-security/)
- [The Little Book of OAuth 2.0 RFCs](https://oauth.net/books/The%20Little%20Book%20of%20OAuth%202.0%20RFCs.pdf)
- [`oauth.net`](https://oauth.net/)
  - [OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified/)
  - [OAuth 2.0 Simplified](https://oauth2simplified.com/)
  - [OAuth 2.0 Servers](https://www.oauth.com/)
- [(Udemy) Master OAuth 2.0: A Practical Guide to API Security](https://www.udemy.com/course/master-oauth-2-api-security-practical-guide)
- [(Udemy) Web Authentication With Golang - Google's Go Language](https://www.udemy.com/course/oauth-authentication/)
- [RFC list](https://www.rfc-editor.org/search/rfc_search_detail.php?title=OAuth&page=All)
- OAuth 2.0 em português claro:
  - [Parte I](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-i-23c5618a4601)
  - [Parte II](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-ii-e8e0d8dba994)
  - [Parte III](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-iii-669a07807f9a)

### Ferramentas para Testar

- Postman: Suporte nativo a fluxos OAuth.
- oauth.tools: Site interativo para simular fluxos.
- Keycloak: Servidor de autorização open-source.

### Mapa das Especificações do OAuth 2.0

> [`https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/`](https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/)

The OAuth 2.0 Core Framework (RFC 6749) defines roles and a base level of functionality, but leaves a lot of implementation details unspecified. Since the publication of the RFC, the OAuth Working Group has published many additional specs built on top of this framework to fill in the missing pieces. Looking at the full list of specs the group is working on can be somewhat overwhelming. This chapter lays out how the various specs relate to each other.

## Cursos

- [The Nuts and Bolts of OAuth 2.0](The%20Nuts%20and%20Bolts%20of%20OAuth%202.0.md)
- [Advanced OAuth Security](./Advanced%20OAuth%20Security.md)
- [OAuth 2.0 School](./OAuth%202.0%20School.md)
- [Master OAuth 2.0: A Practical Guide to API Security](./Master%20OAuth%202.0%20A%20Practical%20Guide%20to%20API%20Security.md)

## That's all

...folks!!!
