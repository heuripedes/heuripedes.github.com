---
layout: post
title: Renomear arquivos MP3 de acordo com as tags ID3 usando o php
---

Ontem baixei o <a href="http://getsongbird.com/">Songbird</a>, é pesadinho mas gostei mesmo sabendo que eu vou acabar voltando para o MPD. Mas aí eu vi um problema  com as músicas: em alguns casos o Songbird não encontrava informações das músicas. Foi aí que decidi ficar até altas horas editando as tags das minhas músicas para que o Songbird pudesse me apresenta-las de forma melhor.

Hoje de manhã, quando fui colocar alguns mp3's que tinha baixado nas suas devidas pastas percebi a bagunça que havia, alguns arquivos estavam nomeados assim: 123.mp3 ou get_song_nightwish_3.Mp3 e haviam outras variações. Foi aí que decidi fazer um pequeno script php para renomear o nome dos arquivos da forma que quero: Artista - Titulo.mp3. Dei uma pesquisada e encontrei as funções que precisava para ler as tags ID3.

O script não sobrescreve o arquivo atual, ele cria uma pasta chamada mp3_output que contém os arquivos renomeados, os arquivos que não possuem tags ID3 não são renomeados, somente copiados para mp3_output. O script também não converte mp3s recursivamente.

Para usá-lo basta ir para  a pasta que contém os arquivos que você deseja renomear e chamar o script. Ah, e você tem que ter compilado o php com a <a href="http://pecl.php.net/package/id3">extensão ID3</a>.

Tchau! ;)

<a href="http://enygmata.site88.net/scripts/mp3rename.php.gz">Download do script</a>
