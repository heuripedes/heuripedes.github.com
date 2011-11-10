---
layout: post
title: Paper: Garis de memória
---

Repostagem do paper que escrevi para a e-zine [Cogumelo Binário](http://cogumelobinario.hashit.org/). 






				    GARIS DE MEMÓRIA

			 Higor Eurípedes (a.k.a. enygmata ou fmul)
			       heuripedes at gmail dot com
				    Outubro de 2011



	Conteúdo
	==============================================================================

	1 - Introdução
	2 - O que são garbage collectors
	2.1 - Alguns conceitos interessantes
	3 - Tipos
	3.1 - Tracing Garbage Collectors
	3.2 - Reference Counting Garbage Collectors
	3.2.1 - Um refcounted GC por dentro
	4 - Citações, links e referências



	1 - Introdução
	==============================================================================

	Na-não, espere! Este não é um texto sobre os nossos mal tratados lixeiros, 
	menos ainda sobre Alzheimer. Estou aqui para falar sobre os Garbage 
	Collectors, ou GCs.

	Neste artigo irei utilizar os termos "bloco" e "objeto", entenda-os como sendo 
	regiões na heap do programa a menos que fique explícito que estou falando de 
	objetos do paradigma de orientação à objetos.



	2 - O que são garbage collectors
	==============================================================================

	Coletores de lixo ou garbage collectors são peças de código que tem a função 
	de liberar recursos que não são mais utilizados pelo programa a.k.a. o lixo do 
	programa. Eles são comumente utilizados no gerenciamento de memória de 
	linguagens de alto nível (principalmente naquelas que não permitem 
	gerenciamento explícito) e por bibliotecas que permitem o compartilhamento de 
	objetos entre programas.

	Os GCs livram o programador de algumas preocupações, como chamar free() menos 
	ou até mais que o necessário (causando erros double free), mas não é garantia 
	de que não ocorrerão vazamentos de memória. Utilizar coletores de lixo também 
	diminui as chances de que ocorram situações onde não existe nenhum ponteiro 
	apontando pra um determinado bloco na heap, os rebeldes dangling pointers.

	Mas, como diz o ditado e o meu colega sigsegv, "rapadura é doce mas não é 
	mole". Nem tudo são flores no mundo dos garbage collectors, o uso de coletores 
	implica em custos pro sistema e o uso incorreto pode causar pausas em 
	programas multitarefa, picos de processamento aleatórios e fragmentação da 
	memória do programa.


	2.1 - Alguns conceitos interessantes
	------------------------------------------------------------------------------

	Quando falamos sobre garbage collectors repetimos sempre o termo "referência".  
	As referências são ligações entre objetos na memória, são ponteiros, e podem 
	ser classificadas quanto a força que exercem sobre a vida do objeto 
	referenciado. As fracas são referências que muitas vezes sequer contem 
	informações sobre o objeto referenciado (são ponteiros "crus") e não previnem 
	que o mesmo seja coletado. Objetos referenciados fortemente não são coletados 
	até que todas as referencias sejam removidas.

	Outros conceitos importantes são "coleta" e "reciclagem", o primeiro implica 
	em liberar a memória e o segundo significa informar o alocador que aquela 
	região está disponível para reuso. A implementação destes dois conceitos vai 
	depender dos requisitos do sistema.


	3 - Tipos
	==============================================================================

	Basicamente, existem duas vertentes entre os coletores de lixo: os tracing 
	garbage collectors (tracing GC) e os reference counting garbage collectors 
	(refcount GC). É possível também encontrar híbridos. Todos eles, porém, exigem 
	que o programador utilize funções de alocação específicas ou que de alguma 
	forma identifique a memória alocada como passível de coleção.


	3.1 - Tracing Garbage Collectors
	------------------------------------------------------------------------------

	Os tracing GCs, em geral, trabalham escaneando a memória em busca de 
	referências que possam ser alvos de coleta. Estes coletores costumam ser 
	cíclicos, pois não coletam os objetos assim que deixam de ser utilizados, mas 
	durante alguma etapa do ciclo de coleta.

	Como o GC se comporta depende do algoritmo utilizado, o mark-sweep, por 
	exemplo, possui três etapas: na primeira o GC identifica as referências; na 
	segunda ele marca os objetos que ainda possuem referências; e, na terceira 
	etapa, o GC coleta todos os objetos que não estão marcados.

	A linguagem Java (por padrão) e as linguagens que são implementadas com a 
	Common Language Infraestructure da .NET Framework utilizam um tipo de tracing 
	GC chamado efêmero ou geracional, este GC técnicas de heurística para 
	encontrar blocos não utilizados e por isso possui um certo nível de não 
	determinismo, ou seja, torna-se difícil prever quando ocorrerá um ciclo. O uso 
	deste tipo de GC pode causar dores de cabeça em sistemas com carga alta como o 
	que aconteceu no site de perguntas e respostas Stack Overflow (veja o link 
	sobre o GC da .NET no fim do artigo).


	3.2 - Reference Counting Garbage Collectors
	------------------------------------------------------------------------------

	Os refcounting GCs são mais humildes que seus primos e utilizam somente 
	contadores de referência para determinar quando um bloco está pronto para ser 
	coletado. A ideia é simples, sempre que um bloco estiver com zero referências,
	ele estará pronto para ser coletado ou reciclado. Diferente dos tracing GCs, 
	os refcounting GCs costumam guardar a informação sobre as referência junto ao 
	bloco que é alocado para o usuário, muitas vezes na forma de cabeçalho. Este 
	tipo de coletor é bastante utilizado em aplicações que necessitam que os 
	recursos sejam liberados o mais cedo possível.

	Uma desvantagem dos coletores baseados em contagem de referência é que eles 
	não lidam com referências cíclicas sem que a complexidade aumente 
	consideravelmente; na linguagem Perl, por exemplo, objetos com referência 
	cíclica só são liberados numa etapa de coleta que ocorre no fim do programa.  
	A solução mais simples nesses casos é utilizar um misto de referências fracas 
	e fortes entre os objetos envolvidos ou simplesmente proibir que este tipo de 
	referência aconteça.

	Além da Perl, as linguagem Python e PHP e as bibliotecas GObject (parte da 
	GLib e base da biblioteca Gtk+) e COM+ (uma biblioteca para IPC nos sistemas 
	Windows) também utilizam contagem de referência para evitar problemas com 
	alocação de recursos.

	3.2.1 - Um refcounted GC por dentro
	- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

	Como dito antes, os blocos alocados pelos coletores possuem um cabeçalho e 
	também um corpo, onde ficam os dados do usuário. O cabeçalho, em geral, tem um 
	tamanho fixo, o suficiente para armazenar o contador de referência.

	  Bloco alocado pelo coletor:
	    +--------------+------------------->
	    | Cabeçalho    | Corpo 
	    +--------------+------------------->
	    ^-Tamanho fixo-^-Tamanho variável-->

	Em C você pode considerar que a estrutura básica do coletor é definida da 
	seguinte forma:

	  typedef struct {
	      size_t refcount; /* Número de referências. */
	  } GCHeader;

	Nota: O corpo do bloco não tem delimitação e por motivos de portabilidade não 
	vamos definir um membro ou estrutura pra ele.  Vale lembrar que é possível 
	criar uma estrutura para representar todo o bloco utilizando arrays de tamanho 
	variável (C99) ou zero-length arrays.

	No momento da alocação, o tamanho do cabeçalho é somado ao tamanho do bloco 
	que o usuário deseja alocar e em seguida o contador é inicializado com o valor 
	um. O ponteiro pro bloco recentemente alocado é movido para a próxima posição, 
	ele agora aponta para o corpo do bloco.

	  /* aloca um bloco e retorna seu corpo */
	  void *gc_alloc(size_t size) {
	      size_t new_size = size + sizeof(GCHeader);
	      GCHeader *block = malloc(new_size);

	      block->refcount = 1;
	      block++;

	      return block;
	  }

	Para facilitar o trabalho, é uma boa ideia criar funções para acessar a região 
	do cabeçalho:
	 
	  /* retorna o cabeçalho do bloco */
	  GCHeader *gc_get_header(void *body) {
	      return ((GCHeader*)body)-1;
	  }

	E agora a implementação das funções que aumentam ou diminuem o numero de 
	referências. A função para aumentar o número de referências não tem mistério, 
	é só pegar o ponteiro pro cabeçalho e somar um à refcount:

	  /* adiciona uma referência ao bloco e retorna um ponteiro para o corpo */
	  void *gc_ref(void *body) {
	      GCHeader *header = gc_get_header(body);

	      header->refcount++;

	      return body;
	  }

	Já a função para diminuir o número de referências tem é um pouquinho mais 
	complicada: precisamos checar se a quantidade de referências chegou a zero e 
	liberar a memória se isso acontecer.

	  /* remove uma referência e retorna um ponteiro para o corpo (ou null caso 
	     não hajam mais referências */
	  void *gc_unref(void *body) {
	      GCHeader *header = gc_get_header(body);

	      header->refcount--;

	      if (header->refcount == 0) {
		  free(header);
		  body = NULL;
	      }

	      return body;
	  }

	É importante notar que a implementação apresentada aqui não é aconselhada para 
	uso em programas multitarefa, pois não utiliza nenhum mecanismo de 
	sincronização ou operações atômicas.



	4 - Citações, links e referências
	==============================================================================

	- In managed code we trust, our recent battles with the .NET garbage collector
	  http://samsaffron.com/archive/2011/10/28/in-managed-code-we-trust-our-recent-battles-with-the-net-garbage-collector

	- Two-Phased Garbage Collection
	  http://perldoc.perl.org/perlobj.html#Two-Phased-Garbage-Collection

	- Arrays of Length Zero
	  http://gcc.gnu.org/onlinedocs/gcc/Zero-Length.html#Zero-Length

	- Object memory management
	  http://developer.gnome.org/gobject/stable/gobject-memory.html

	- The Memory Management Reference: Beginners Guide: Recycling
	  http://www.memorymanagement.org/articles/recycle.html



