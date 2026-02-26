# Sk1ttl3 News - SQL Injection

###### Solved by @milelaraia

Este desafio está ligado à vulnerabilidades em Banco de Dados dentro de um site.

## Descrição do Desafio

Um mídia que foi demitido do Jornal de Night City decidiu criar seu próprio jornal online. Apesar de estar ganhando fama, ele disse que tem que lidar frequentemente com hackers derrubando sua plataforma ou fazendo postagens falsas. Ele precisa de um trilha-rede com bom conhecimento técnico para testar a segurança do seu jornal. Seu objetivo é acessar o painel administrativo. Boa sorte.

## Análises no Burp Suite

Ao fazer uma pesquisa qualquer no site observamos que as notícias são acessadas via parâmetro `slug`, por exemplo, o parâmetro `/?q=maxtac`. Além disso temos o seguinte comentário: 

[![imagem-2026-02-25-174403478.png](https://i.postimg.cc/bY9SVPVy/imagem-2026-02-25-174403478.png)](https://postimg.cc/PNNqNgk9)

O que significa que há um possível parâmetro `/admin.php` que nos dá acesso ao painel de administrador.

Percebemos também que, ao acessar a página de login `/login.php`, encontramos outro comentário no qual informa que o banco de dados utilizado pelo desenvolvedor foi o `SQLite`.

[![imagem-2026-02-25-174707604.png](https://i.postimg.cc/ydYCVFKr/imagem-2026-02-25-174707604.png)](https://postimg.cc/hJw5sQf9)

## Identificação da Vulnerabilidade

Ao testar o parâmetro `slug` com um payload clássico de `SQL Injection`, percebemos que o site passou a retornar todas as notícias, confirmando uma vulnerabilidade.

```bash
?slug=foo' OR '1'='1' --
```
[![Captura-de-tela-2026-02-25-174919.png](https://i.postimg.cc/Z5GrtcrM/Captura-de-tela-2026-02-25-174919.png)](https://postimg.cc/LJVqjjQt)

Isso indica que a query provavelmente é algo semelhante a:
```bash
SELECT * FROM posts WHERE slug = '$slug'
```
A injeção transforma a query em:
```bash
SELECT * FROM posts WHERE slug = 'foo' OR '1'='1'
```
Como `'1'='1'` é sempre verdadeiro, todas as linhas são retornadas.

## Exploração do Login

Quando tentamos acessar o parâmetro `/admin.ph` o site nos retorna o `/login.php`, o que significa que o usuário é validado como `admin` somente após a autenticação.

Sabendo que o sistema utiliza `SQLite` e já possui SQL Injection confirmada, o próximo passo foi testar o formulário de login com o seguinte `payload`:

```bash
Usuário: admin
Senha: ' OR '1'='1' --
```
Considerando que a query de login provavelmente seja:

```bash
SELECT * FROM users 
WHERE username='$username' 
AND password='$password'
```

Após o payload será retornado:

```bash
SELECT * FROM users 
WHERE username='admin' 
AND password='' OR '1'='1'
```

Devido à precedência dos operadores, a condição '1'='1' torna a cláusula WHERE verdadeira, permitindo autenticação sem a senha correta.

[![imagem-2026-02-25-180347157.png](https://i.postimg.cc/m2cwvhv0/imagem-2026-02-25-180347157.png)](https://postimg.cc/2VDhvj8w)

Dessa maneira, acessamos o painel de administrador e conseguimos a `flag`.

[![imagem-2026-02-25-180455119.png](https://i.postimg.cc/HLTbkLh9/imagem-2026-02-25-180455119.png)](https://postimg.cc/YvyvXtkv)

`Flag: DUCK{SK1TTL3_N3W5_2077_SQL1NJ3CT10N_H45_B33N_H4CK3D}`

## Conclusão e Mitigações

Este desafio demonstrou uma vulnerabilidade clássica de SQL Injection, causada pela ausência de validação e uso de consultas parametrizadas no banco SQLite. A falha permitiu manipular a lógica da aplicação e realizar authentication bypass, obtendo acesso ao painel administrativo sem credenciais válidas. No contexto do mundo real, esse tipo de vulnerabilidade é extremamente crítico, pois pode levar ao vazamento de dados sensíveis, escalonamento de privilégios e comprometimento total do sistema. O desafio reforça a importância de práticas seguras de desenvolvimento, como consultas parametrizadas, validação adequada de entradas e controle rigoroso de autenticação e sessão.
