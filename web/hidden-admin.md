# [Hidden Admin](https://shellterlabs.com/pt/training/get-started/web-application-basics/questions/hidden-admin/)
**Descrição**: `Ache a página escondida do administrador. Tente ver o seu conteúdo sem saber o usuário e senha.`


# Levantamento de informações

Ao iniciarmos o desafio, ele nos retorna para uma página principal de um sistema  

> Digital chipmunks

observando a estrutura do código fonte, nada útil nos é retornado.

Ao buscarmos pelo arquivo "robots.txt", arquivo que funciona como filtro para os robôs dos sites de busca, permitindo ou não à exibição de determinado conteúdo, vemos que o arquivo está ativado, e vemos que ele está bloqueando algumas páginas:

`User-Agent: *`
`Disallow: /admin-login`
`Disallow: /css`
`Disallow: /img`

Ao acessarmos a página **/admin-login**, somos redirecionados de volta à um diretório idêntico à principal, porém, com a seguinte mensagem:

*`log in to acess the admin page`*

## Resolvendo o problema

Ao observar os cookies das requisições às paginas, podemos observar que a mesma possui um cookie de nome que nos é interessante:
	
    curl --head url_do_desafio
`url_do_desafio= labs.shellterlabs.com:[sua_porta]`

![is_admin_cookie](https://i.imgur.com/QRv0cRN.png)

**is_admin=false**
ao alterarmos o valor do cookie para true:

    curl -H "Cookie: is_admin=true" -i http://labs.shellterlabs.com:[sua_porta]
Heis a flag!

**`FLAG: SHELLTER{EVERYTHINGNEEDSASTART}`**
-