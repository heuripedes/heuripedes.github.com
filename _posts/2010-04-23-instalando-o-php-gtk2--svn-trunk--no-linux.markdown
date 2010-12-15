---
layout: post
title: Instalando o php-gtk2 (svn trunk) no Linux
---
Com a mudança do repositório do php para svn e com o [lançamento da extensão Cairo pra php] [lancamento], meu tutorial anterior sobre como instalar o php-gtk2 a partir do repositório CVS se tornou obsoleto, mas vou deixá-lo no blog caso alguém ainda precise dele. Aqui vai a receita nova!

Ingredientes
---

* Uma porção grande de Linux
* Uma xícara de PHP versão 5.2 ou superior (estou usando 5.3.0)
* Uma colher de Gtk versão 2.8 ou superior (recomendo 2.12 no minimo)
* Uma porção de GCC, Make, Autoconf e Automake
* Uma pitada de Svn

Modo de preparo
---

Primeiro precisamos baixar o código fonte do php-gtk e da extensão cairo, então dirija-se ao diretório mais próximo e adicione o código que está no SVN aos poucos (pode demorar dependendo da sua conexão):
	
	$ svn co http://svn.php.net/repository/gtk/php-gtk/trunk php-gtk
	$ svn co http://svn.php.net/repository/pecl/cairo/trunk cairo
		
Entre no diretório da extensão cairo e compile bem.

	$ cd cairo	
	$ phpize
	$ ./configure &amp;&amp; make
	$ sudo make install</code>
		
O cairo está pronto, agora temos que preparar o php-gtk.
		
*Atualização*: A partir da [revisão 289265] [revisao] não é mais necessário adicionar manualmente a flag HAVE_CAIRO à CFLAGS.
		
	$ cd ..
	$ cd php-gtk
	$ ./buildconf
	$ ./configure
	$ sudo make install
		
Yum! Isto parece delicioso, vamos provar? Vá ao diretório do php-gtk (se ainda não estiver) e execute o seguinte comando:
		
	$ php -d extension=cairo.so -d extension=php_gtk2.so demos/stock-browser.php
		
Uma janela listando os items do GtkStock deve aparecer.
		
<a href="http://img190.imageshack.us/i/stockbrowser.jpg/" target="_blank"><img src="http://img190.imageshack.us/img190/2991/stockbrowser.th.jpg" alt="Free Image Hosting at www.ImageShack.us" border="0"></a>
		
Então é isso, espero que a receita tenha dado certo. Qualquer coisa é só colocar um comentário. Tchau! :)
		
OBS: Não testei mas é provável que o tutorial funcione para outros ambientes Unix-like.

[lancamento]: http://elizabethmariesmith.com/cairo-alpha-released "Cairo Alpha Released"
[revisao]: http://svn.php.net/viewvc?view=revision&revision=289265

