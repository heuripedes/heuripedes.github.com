---
layout: post
title: Sobre arrays mágicas
---

Se você programa em C a algum tempo, já deve ter ouvido falar nas [arrays de 
tamanho variável][1]. Este tipo de array se tornou padrão em 1999 mas já era 
suportado anteriormente pelo GCC por meio de uma extensão. O trecho abaixo 
é um exemplo de uso/definição de arrays de tamanho variável:

{% highlight cpp %}
void junta_e_imprime(char *a, char *b) {
	char saida[strlen(a) + strlen(b) + 1];
	strcpy(saida, a);
	strcpy(saida, b);

	printf("%s\n", saida);
}
{% endhighlight %}

Mas este post é sobre um tipo de array de tamanho variável que fica dentro 
de estruturas. Por não saber o nome específico deste tipo de array, vou 
chamá-lo de _array mágica_.

> Eu poderia utilizar o nome [zero-length arrays][], mas neste post estou 
> demonstrando uma forma mais portável deste tipo de array por utilizar o 
> tamanho 1.

As arrays mágicas são bastante utilizadas em estruturas que representam 
arquivos em um determinado formato, pois permitem um melhor mapeamento de 
regiões do arquivo. Há também aqueles que utilizam esta técnica para 
diminuir o número de alocações e desalocações.

Para exemplificar como estas arrays funcionam, imagine um formato de 
arquivo de texto em que os 4 primeiros bytes são destinados ao tamanho do 
texto (em bytes) e todo o resto do arquivo é o próprio texto:

	+--------------------+
	| 4 bytes - Tamanho  |
	+--------------------|
	| n bytes - Conteúdo |
	+--------------------+

Uma implementação comum provavelmente utilizaria ponteiros ordinários:

{% highlight cpp %}
struct textfile {
	int32_t size;  /* tamanho */
	char *content; /* texto */
}
{% endhighlight %}

Esta implementação possui dois possíveis problemas ao ambiente onde será 
executada:

- Desperdício de no mínimo 4 bytes com o ponteiro `content`.
- Alocação desnecessária de um bloco extra de memória para o conteúdo de `content`

> Se você nunca programou para dispositivos embarcados deve está pensando:
> "Que cara doente! São só míseros 4 bytes ou uma alocação a mais!"
> Mas se você executar a mesma rotina 10 vezes já serão 40bytes e 10 
> alocações a mais. Depois de uns dois anos seu código pode ter desperdiçado 
> uns 8GB de memória e um mês em chamadas à `malloc()` e `free()`.

Utilizando as tais arrays mágicas, você supera os dois problemas. Com uma 
única alocação você terá espaço para a array e para os outros campos da 
estrutura e não vai precisar de ponteiros comuns:

{% highlight cpp %}
struct textfile {
	int32_t size;    /* tamanho */
	char content[1]; /* texto */
}
{% endhighlight %}

A "mágica" na alocação desta estrutura está em utilizar como tamanho do 
campo `content` o tamanho do conteúdo que ele terá de suportar. O tamanho 
total do requisitado ao `malloc()` será a soma do tamanho dos membros 
anteriores à `content` com o tamanho do conteúdo a ser colocado dentro de 
`content`. Você pode utilizar uma sequência de `sizeof()`s ou então a palavra 
chave `offsetof()`:

{% highlight cpp %}
struct textfile f;
int32_t content_size;

fread(&content_size, sizeof(int32_t), 1, fp);

f = malloc(offsetof(struct textfile, content) + (sizeof(char) * content_size));
f->size = content_size;

fread(f->content, sizeof(char), f->size, fp);
{% endhighlight %}

O acesso ao conteúdo da array é feito normalmente,  acessar o elemento 30,
por exemplo, será válido desde que `content` possua no mínimo 31 elementos.

A técnica funciona porque na linguagem C não há [checagem de limites][2] ou 
limites internos aos blocos de memória do programa. De fato, até algo como 
`f->content[-1]` pode ser considerado válido pois você estará acessando a 
àrea da memória onde está alojado o conteúdo do membro `size`; já 
`f->content[-10]` poderá resultar em falha de segmentação se o bloco de 
memória anterior ao bloco apontado por `f` não pertencer ao programa atual.

Have Fun. :-)

[1]: http://gcc.gnu.org/onlinedocs/gcc/Variable-Length.html 
	"Variable-length arrays"
[2]: https://secure.wikimedia.org/wikipedia/en/wiki/Bounds_checking
	"Bounds checking"

[zero-length arrays]: http://gcc.gnu.org/onlinedocs/gcc/Zero-Length.html
	"Zero-length arrays"

