---
layout: post
title: Convertendo string multibyte UTF-8 para wchar_t e vice-versa com iconv
---

Nos ultimos dias eu estive trabalhando em uma forma de suportar  caracteres utf-8 no bitsheet. Ontem eu resolvi usar a iconv para fazer  este trabalho de conversão de wchar_t para UTF-8 e vice-versa. Resolvi  postar aqui a solução que usei.

	<Teoria chata>

O tipo wchar_t foi introduzido para que houvesse suporte a wide  characters, mas a codificação de uma array de wchar_t (ou wide string) é  dependente da locale do sistema (LC_TYPE/LC_ALL/LC_LANG) e do  compilador. O padrão C não especifica um tamanho exato para o tipo  wchar_t, largando esta tarefa aos compiladores, então o valor deste tipo  geralmente varia entre 8 e 32 bits. No caso da implementação do wchar_t  do gcc, o tamanho é 32 bits, portanto o mesmo comporta caracteres que  consumam até 4bytes (como alguns caracteres Unicode).

UTF-8 (8-bit Unicode Transformation Format) é uma das primeiras  codificações que implementam suporte a Unicode characters, ele foi  criado por Ken Thompson (o cara que criou a linguagem B e que ja recebeu  até premio turing junto com o Denis Ritchie) e Rob Pike (junto com Ken  Thompson criou o sistema operacional Plan 9) num restaurante de New  Jersey (pode acreditar!) em setembro de 1992. Os caras resolveram fazer  isso porque odiaram o suporte a caracteres 16bit do UTF original e do  ISO 10646. O formato foi feito para ser compatível com o padrão ASCII e  utiliza um numero variado de bytes para representar os caracteres  (caracteres ASCII usam 1 caracteres, e letras acentuadas geralmente  utilizam 2 caracteres).

Nem todas as implementações do tipo wchar_t comportam todas as  possibilidades da codificação UTF-8, o wchar_t do compilador c++ da  microsoft é de 16bits e portanto não comporta os casos em que os  caracteres utf-8 são maiores que 16bits (É UMA CILADA BINO!). Mas aí  existem umas manhas pra resolver este problema.

	</Teoria chata>

A biblioteca iconv é uma biblioteca comum em sistemas UNIX, ela faz a  conversão entre diferentes codificações de caracteres inclusive de uma  codificação como o utf-8 para a codificação interna do wchar_t. Chega de  enrolação, la vai o codigo.

(a linguagem é C++)
{% highlight cpp %}
// lembre se desses headers:
#include <cwchar>
#include <iconv.h>
#include <langinfo.h>

// UTF-8 multibyte string para wide char string
wchar_t * utf8mbstowcs (const char *str) {
	iconv_t cd;
	size_t inleft, outleft;
	wchar_t *wbuf;
	wbuf = new wchar_t[strlen(str)+1];
	memset(wbuf, 0, strlen(str)+1);

	cd = iconv_open("WCHAR_T", "UTF8");
	iconv(cd, NULL, NULL, NULL, NULL);

	inleft  = strlen(str) + 1;
	outleft = strlen(str) * sizeof(wchar_t)+1;

	char *in  = (char*)str;
	char *out = reinterpret_cast<char *>(wbuf);
	
	if (iconv(cd, &in, &inleft, &out, &outleft) < 0) {
		delete wbuf;
		wbuf = NULL;
	}

	iconv_close(cd);
	return wbuf;
}
 
// wide char string para UTF-8 multibyte string
char *wcstoutf8mbs (const wchar_t *wstr) {
	iconv_t cd;
	size_t inleft, outleft;
	char *buf;
	char str[wcstombs(NULL, wstr, 0)+1];
	inleft  = wcstombs(NULL, wstr, 0)+1;
	outleft = (inleft -1)* sizeof(wchar_t)+1;
	memset(str, 0, inleft);
	
	// melhor converter, vai que algo errado acontece por conta
	// de endianess
	wcstombs(str, wstr, inleft); 
	buf = new char[outleft];
	memset(buf, 0, outleft);
	
	cd = iconv_open("UTF8", nl_langinfo(CODESET));
	iconv(cd, NULL, NULL, NULL, NULL);
	
	char *in  = reinterpret_cast<char *>((wchar_t*)str);
	char *out = (char*) buf;
	
	if (iconv(cd, &in, &inleft, &out, &outleft) < 0) {
		delete buf;
		buf = NULL;
	}

	iconv_close(cd);
	return buf;
}
{% endhighlight %}

Referências:

* http://www.cl.cam.ac.uk/~mgk25/ucs/utf-8-history.txt
* http://opengroup.org/onlinepubs/007908799/xsh/wchar.h.html
* http://icu-project.org/docs/papers/unicode_wchar_t.html
