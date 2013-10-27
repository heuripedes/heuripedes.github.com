---
layout: post
title: Cuidado com o cabeçalho Host do HTTP
---

A especificação da versão 1.1 do protocolo HTTP descreve um cabeçalho chamado
Host, este cabeçalho contém o nome de host do site que o usuário deseja
acessar. O cabeçalho serve para que o servidor web possa identificar qual
recurso o usuário está requisitando quando o servidor atende múltiplos
nomes de host (sites).

Apesar de exigir a presença do cabeçalho, a RFC do protocolo, no [item 5.2][1],
autoriza que o mesmo seja ignorado caso o cliente HTTP faça a requisição
utilizando uma [URI absoluta][2]. Abaixo você pode ver um exemplo de requisição
com URI relativa e outro com URI absoluta, respectivamente:

	GET /index.php HTTP/1.1
	Host: exemplo.com.br


	GET http://exemplo.com.br/index.php HTTP/1.1
	Host: exemplo.com.br

Ignorar o valor Host não representa um problema de segurança, exceto quando o
programador assume que ele sempre representa o servidor atual. Infelizmente
isso acontece bastante, muitos colocam códigos como o abaixo para diferenciar
entre modo de produção e modo de desenvolvimento em seus programas:

{% highlight php5 %}
if (preg_match('/^(localhost|127.0.0.1)(\:\d+)?/', $_SERVER['HTTP_HOST'])) {
	imprime_algo_comprometedor();
}
{% endhighlight %}

Para explorar a falha de segurança, é necessário que você crie uma requisição
HTTP customizada que envia o host correto pela URI mas enviar `localhost` no
cabeçalho `Host`. Podemos fazer isso usando telnet:

	[higor@whiterice tmp]$ telnet exemplo.com.br 80
	Trying 127.0.0.1...
	Connected to exemplo.com.br
	Escape character is '^]'.
	GET http://exemplo.com.br/index.php HTTP/1.1
	Host: localhost

	HTTP/1.1 200 OK
	Date: Sun, 27 Oct 2013 16:11:42 GMT
	Server: Apache
	Connection: close
	Transfer-Encoding: chunked
	Content-Type: text/html

	MODO DE DESENVOLVIMENTO. A SENHA DE admin É "123tr0c4r!"

	Connection closed by foreign host.

E assim nós descobrimos a senha de `admin`.

Se você quer evitar este tipo de problema, você deve utilizar o nome do
servidor (`$_SERVER['SERVER_NAME'])` em vez do valor de Host
(`$_SERVER['HTTP_HOST']`). Porém ainda existe algo fazer se você utiliza o
Apache para servir suas páginas, será necessário dizer ao Apache que você quer
que a variavel de ambiente `SERVER_NAME` seja igual ao nome que você deu ao seu
servidor (`ServerName`), pra isso você deve ativar a opção `UseCanonicalName`:

{% highlight apacheconf %}
<VirtualHost *>
	ServerName exemplo.com.br
	UseCanonicalName on
</VirtualHost>
{% endhighlight %}

Se você utiliza PHP e/ou Apache, você pode ler mais sobre a solução e sobre o
problema [neste resposta do StackOverflow](http://stackoverflow.com/a/2297421).


[1]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.2 "RFC 2616 - 5.2 - The Resource Identified by a Request"
[2]: http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html#sec5.1.2 "RFC 2616 - 5.1.2 - Request-URI"

