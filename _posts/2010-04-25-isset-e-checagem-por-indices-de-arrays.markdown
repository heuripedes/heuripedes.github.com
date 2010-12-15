---
layout: post
title: isset e checagem por indices de arrays
---
Muita gente se depara com erros de lógica pelo mal uso da função isset(). O que ocorre é que geralmente queremos que uma ação seja feita caso a variável não esteja definida, porém se definirmos a variável como null a função isset irá retornar false como se a variável não tivesse sido definida.

O mesmo ocorre com a checagem por índices de arrays, caso o valor para o qual o índice aponta estiver definido como null a função isset irá retornar false. A solução para este problema é utilizar a função array_key_exists() caso o valor null numa determinada posição de array seja importante para sua aplicação. O código abaixo demonstra o comportamento das duas funções.

{% highlight php %}
<?php

$a = array('key' => null);
if (!isset($a['key'])) {
	print "isset: o indice 'key' não existe em \$a!\n";
}

if (!array_key_exists('key', $a)) {
	print "array_key_exists: o indice 'key' não existe em \$a!\n";
}
{% endhighlight %}
Saída:

	isset: o indice 'key' não existe em $a!


