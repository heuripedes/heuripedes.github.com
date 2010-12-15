---
layout: post
title: Instalando o PHP-GTK2 mais recente no Linux
---

Olá, hoje estou aqui para ensinar como instalar o PHP-GTK do CVS no Linux. Resolvi fazer este tutorial porque vejo que muita gente está tendo problemas com isso, e também por que eu quero te-lo anotado em algum lugar hehehe. Bem, mãos a obra!.

Antes de começarmos, certifique-se que você possui o php compilado e instalado na sua maquina (isso mesmo, você vai ter que pegar o source e compilar, presumo que você ja saiba fazer isso). Veja também se você possui as seguintes bibliotecas e seus cabeçalhos:

* Gtk versão >= 2.14
* LibCairo versão >= 1.8
* Glib versão >= 2.14

Versões anteriores funcionam, porém estas são as necessárias para que você tenha suporte completo (ou quase) das funções Gtk.

Agora devemos baixar o source do php-gtk que está no repositorio do php, para isso vamos primeiro logar como 'cvsread', a senha é 'phpfi':

	$ cvs -d :pserver:cvsread@cvs.php.net:/repository login

Agora podemos baixar o source do php e do php-gtk do repositório rodando este comando:

	$ cvs -d :pserver:cvsread@cvs.php.net:/repository co php-gtk

O php-gtk do cvs requer a extensão php-cairo se você está compilando num sistema que possui gtk superior a 2.8, então vamos baixá-lo também!

	$ svn co svn://whisky.macvicar.net/php-cairo

Agora compilamos o php-cairo com os seguintes comandos:

	$ cd php-cairo
	$ phpize
	$ ./configure && make
	$ sudo make install

OBS: caso ocorra um problema quado você rodar o ./configure, antes de rodá-lo execute o autoconf mais recente que você possuir em sua máquina (no ubuntu  8.10 é o programa autoconf2.50) e depois rode

	$ ./configure && make
	$ sudo make install.

Finalmente, a hora está chegando... vamos compilar o php-gtk!!  Vá para o diretório do php-gtk e execute estes comandos:

	$ ./buildconf
	$ CFLAGS="-DHAVE_CAIRO" ./configure
	$ make
	$ sudo make install

TADAA! Teu php-gtk deve estar funcionando uma hora dessas :) Huh? Quer testar?? Então rode a seguinte linha de comando:

	$ php -d extension=cairo.so -d extension=php_gtk2.so -r '$w=new GtkWindow();$l=new GtkLabel("Hello");$w->add($l);$w->show_all();$w->connect_simple("destroy", array("gtk", "main_quit"));$w->set_size_request(300,300); gtk::main();'</code>

Po-por hoje é só pe-pessoal!
