# Tanuki Bet - Exploração Web

###### Solved by @milelaraia

Este desafio está diretamente ligado à exploração de vulnerabilidades de um site web que foi criado exclusivamente para esse propósito.

## Primeiras Análises

- Importante: Todas as análises foram feitas com a ferramenta de exploração `Burp Suite` e com uma extensão da mesma,  `JWT editor`.

Ao criar uma conta no site e observar-la pelo Burp, podemos ver que toda conta tem um "número" específico de `user` e que o `token JWT` é o mesmo para todas as contas.

[![imagem-2026-02-11-151442233.png](https://i.postimg.cc/fWx9BZb5/imagem-2026-02-11-151442233.png)](https://postimg.cc/7bZhZF0T)

Dessa maneira podemos percorrer por alguns dados de outros users. 

[![imagem-2026-02-11-152005129.png](https://i.postimg.cc/qMNdpBCY/imagem-2026-02-11-152005129.png)](https://postimg.cc/w7YrVp2c)

Além disso ao digitar um caminho inexiste de url ele nos dá acesso à uma lista de caminhos possíveis.

[![imagem-2026-02-11-152239506.png](https://i.postimg.cc/tCFGsS2n/imagem-2026-02-11-152239506.png)](https://postimg.cc/zVDcMjmq)

## Domínios de Interesse

Ao acessar o domínio `api/admin/secret` nos é retornado uma mensagem de token `inválido`.

[![imagem-2026-02-11-152533564.png](https://i.postimg.cc/8cBwS6mP/imagem-2026-02-11-152533564.png)](https://postimg.cc/K3jt74Wd)

Vamos então inserir um token válido por meio da ferramenta `Repeater`. Lembrando que o token é sempre o mesmo, então vamos pegar o de usuário.

```bash
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxOCwidXNlcm5hbWUiOiJjYXJsb3MiLCJyb2xlIjoidXNlciIsInNlY3JldF9ub3RlIjoiRkxBR3tqd3RfczNuczF0MXYzX2Q0dDRfMW5fcDR5bDA0ZH0iLCJleHAiOjE3NzA5MTk3ODIsImlhdCI6MTc3MDgzMzM4Mn0.ZaTr3pxJC_Oci0vsTT10eXej51iQmU2PsR6t0cE3gq8
```
[![imagem-2026-02-11-153041066.png](https://i.postimg.cc/fTSTBf54/imagem-2026-02-11-153041066.png)](https://postimg.cc/hfgBGxHM)

Ele nos retorna `Acesso Negado`. Porém, ao analisar os usuários anteriormente, encontramos que o `user 1` é o nosso `admin`.

[![imagem-2026-02-11-153333214.png](https://i.postimg.cc/FKCHTCt3/imagem-2026-02-11-153333214.png)](https://postimg.cc/FfcvRZHH)

## Acessando a vulnerabilidade

Utilizando a extensão citada anteriormente, conseguimos editar o nosso `token JWT`. Primeiramente iremos modificar a codificação atual dele de `HS256` para `none`, deixando o token sem assinatura e tornando possível a edição.

[![imagem-2026-02-11-153852495.png](https://i.postimg.cc/PxH2Fy1c/imagem-2026-02-11-153852495.png)](https://postimg.cc/sBnYMp69)

Bom, como já temos as credenciais do nosso administrador e podemos editar o JWT agora é só finalizar:

[![imagem-2026-02-11-154328994.png](https://i.postimg.cc/rF5N0b3t/imagem-2026-02-11-154328994.png)](https://postimg.cc/vcHVCPQQ)

Dessa maneira, temos a nossa flag: `FLAG{jwt_w34k_s3cr3t_cr4ck3d}`.

