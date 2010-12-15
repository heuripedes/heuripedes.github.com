---
layout: post
title: Integrando Doxygen ao GNU Build System
---

Depois de procurar muito e só achar coisas que não funcionavam, eu resolvi meter a mão na massa! Foi até divertido!

Como todos sabem, ou deveriam saber, o GNU Build System - autohell para os mais íntimos - é o sistema de compilação mais famoso no mundo UNIX, ele tem suporte padrão a algumas linguagens (como C, C++ e Fortran), compilação de documentação na forma de man pages e LaTeX, e até uma suite de testes - essa eu não sei nem pra onde vai.

Até aí tudo bem com o autohell, o problema é que eu não queria gerar man pages e muito menos documentos TeX: eu queria HTML pra ficar bonitinho que nem aquelas documentações de gente que sabe programar. Foi aí que encontrei o Doxygen, ele gera HTML e também tem suporte a uma pancada de formatos. O que eu queria mesmo era poder gerar a documentação usando o próprio autohell, e aí eu saí procurando e acabei achando algumas soluções como uma que está no arquivo de macros do autoconf. Só que a unica coisa que elas estavam fazendo era procurar o doxygen e outras ferramentas que ele precisa no sistema. O aminclude de algumas soluções que encontrei, estava muito mal escrito e o make relatava problemas o tempo todo.

Bom, vamos deixar de enrolação e por logo a mão na massa. Primeiro, no configure.ac, você tem que adicionar o código a seguir:

{% highlight bash %}
# Checa se o executavel doxygen existe no $PATH, se existir $DOXYGEN
# recebe doxygen
AC_CHECK_PROG([DOXYGEN], [doxygen], [doxygen], [])

# Agora a gente faz a checagem pra ver se o doxygen foi detectado
# mesmo ou não se ele tiver sido detectado, temos que adicionar alguns
# alvos para o make
AC_MSG_CHECKING([wheter to add documentation targets])
if test ! -z "$DOXYGEN"; then
	AC_MSG_RESULT([yes])
else
	AC_MSG_RESULT([no])
fi

# Se o doxygen foi detectado DOXY_DOC vai ser true.
AM_CONDITIONAL([DOXY_DOC],[test ! -z "$DOXYGEN"])

# A variavel DOXYGEN é só para o make encontrar o executavel do dito
# cujo.
AC_SUBST([DOXYGEN], [$DOXYGEN])

AC_CONFIG_FILES([Makefile doc/Doxyfile doc/Makefile outro_diretorio/Makefile])
{% endhighlight %}

No Makefile.am da raiz do seu projeto você só precisa colocar o seguinte:

{% highlight makefile %}
# No projeto em que estou trabalhando o diretório da documentação é o
# diretorio doc, por isso DOC_DIR = doc quando o doxygen é
# detectado
DOC_DIR =
if DOXY_DOC
DOC_DIR = doc
endif
SUBDIRS = $(DOC_DIR) outro_diretorio

# Aqui a gente adiciona o alvo para o make
doc:
cd $(top_srcdir)/doc && $(MAKE) && cd ..

# E aqui a gente diz ao make que o alvo doc não pode ser tratado como
# arquivo.
.PHONY: doc
{% endhighlight %}

O diretório da documentação também precisa de um Makefile.am, é esse o arquivo responsável por executar a compilação da documentação.

{% highlight makefile %}
# Isso aqui faz o make compilar quando ele for executar o make all.
all: html-local

# Instruções para compilação da documentação
html-local:
	if DOXY_DOC
	rm -rf html/
	$(DOXYGEN) 
	else
	@echo Doxygen not detected, skipping documentation build.
	endif

# Arquivos a serem copiados
html_DATA = html/*

# Truque para o make não reclamar quando não achar os alvos com nomes 
# dos arquivos gerados pelo Doxygen
$(html_DATA):

# Instruções para instalação da documentação
install-html-local:
	@$(NORMAL_INSTALL)
	test -z "$(htmldir)" || $(MKDIR_P) "$(DESTDIR)$(htmldir)"
	@list='$(html_DATA)'; \
	for p in $$list; do \
		if test -f "$$p"; then d=; else d="$(srcdir)/"; fi; \
		f=$(am__strip_dir) \
		echo " $(INSTALL_DATA) '$$d$$p' '$(DESTDIR)$(htmldir)/$$f'"; \
		$(INSTALL_DATA) "$$d$$p" "$(DESTDIR)$(htmldir)/$$f"; \
	done

# Instruções para limpeza de arquivos.
clean-local:
	-rm -rf html/
{% endhighlight %}

{% highlight makefile %}
# Variáveis com os nomes de arquivos a ser excluidos
CLEANFILES = Doxyfile Makefile.in
MAINTAINERCLEANFILES = $(CLEANFILES)

{% endhighlight %}
Para finalizar, você precisa ter um arquivo Doxyfile.in no diretório da documentação. Ahn? Por que você precisa? Pelo simples fato de que é melhor usar paths absolutas para determinar o caminho do diretório onde está o source e do de saída da documentação. Assim evitamos problemas caso desejemos dar um make all dentro do diretório da documentação. No Doxyfile você precisa mudar o valor de algumas variáveis como no exemplo abaixo:

	PROJECT_NAME           = "@PACKAGE_NAME@"
	PROJECT_NUMBER         = "@PACKAGE_VERSION@"
	OUTPUT_DIRECTORY       = @abs_top_srcdir@/doc
	STRIP_FROM_PATH        = @abs_top_srcdir@

Uma vantagem de se fazer isso é não precisar ficar atualizando o nome do projeto ou a versão, toda vez que você resolver mudar.

Até a próxima.

