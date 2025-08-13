# sobreOAuth

## Sobre sobreOAuth

Estudo e compilação sobre o protocolo OAuth.

> ***Alguns termos serão mantidos em inglês.***

## Referências

- [(Udemy) The Nuts and Bolts of OAuth 2.0](https://www.udemy.com/course/oauth-2-simplified)
- [(Udemy) Advanced OAuth Security](https://www.udemy.com/course/advanced-oauth-security/)
- [The Little Book of OAuth 2.0 RFCs](https://oauth.net/books/The%20Little%20Book%20of%20OAuth%202.0%20RFCs.pdf)
- [OAuth 2 Simplified](https://aaronparecki.com/oauth-2-simplified/)
- [(Udemy) Master OAuth 2.0: A Practical Guide to API Security](https://www.udemy.com/course/master-oauth-2-api-security-practical-guide)
- [(Udemy) Web Authentication With Golang - Google's Go Language](https://www.udemy.com/course/oauth-authentication/)
- [RFC list](https://www.rfc-editor.org/search/rfc_search_detail.php?title=OAuth&page=All)
- OAuth 2.0 em português claro:
  - [Parte I](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-i-23c5618a4601)
  - [Parte II](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-ii-e8e0d8dba994)
  - [Parte III](https://erikshimoda.medium.com/oauth-2-0-em-portugu%C3%AAs-claro-parte-iii-669a07807f9a)

## Mapa das Especificações do OAuth 2.0

> `https://www.oauth.com/oauth2-servers/map-oauth-2-0-specs/`

The OAuth 2.0 Core Framework (RFC 6749) defines roles and a base level of functionality, but leaves a lot of implementation details unspecified. Since the publication of the RFC, the OAuth Working Group has published many additional specs built on top of this framework to fill in the missing pieces. Looking at the full list of specs the group is working on can be somewhat overwhelming. This chapter lays out how the various specs relate to each other.

## Terminologia

Regras:

- Resource owner
- Client
  - Confidential clients (no servidor)
  - Public clients (browser web ou aplicativos móveis)
- Resource server
- Authorization server

Tokens:

- Authorization code
- Access token
- Refresh token

Grant types (a.k.a. Grant flows ou Authorization flows):

- Authorization Code: get auth code and exchange it for access token
- Client Credentials: for service-to-service communication (machine-to-machine flow)
- Device Code: for devices with limited input (browseless or input constraint devices)
- Refresh Token: for getting access token without asking user to login again
- Implicit Flow (legacy): get access token directly without auth code
- Password Grant (legacy): ask user for password

Scope:

Conjunto de informações que serão expostas pelo token.
