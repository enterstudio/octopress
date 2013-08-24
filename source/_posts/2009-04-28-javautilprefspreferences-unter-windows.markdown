--- 
layout: post
title: java.util.prefs.Preferences unter Windows
categories: 
- java
- software-entwicklung
status: publish
type: post
published: true
---
Nutze ich Windows, fühle ich mich immer etwas eingeschränkt und alles geht etwas langsamer. Diesmal jedoch bin ich sicher, dass es nicht mein Fehler war :-)

Während ich meine Diplomverteidigung vorbereitete kam ich in die Situation, dass ich unter Windows (Mein Freund [Benni][1] war so nett mir sein Netbook zur Präsentation zur Verfügung zu stellen) die von [JMonkeyEngine][2] genutzten Voreinstellungen zu löschen. Einmal an der falschen Stelle geklickt und zack, meine Software startete immer in der falschen Auflösung. JmonkeyEngine nutzt für seine Voreinstellungen die Klasse [java.util.prefs.Preferences][3]. Das macht ja soweit auch Sinn, dafür ist sie ja da. In den Javadocs steht auch, dass diese Klasse die Voreinstellungen mit den vom jeweiligen Betriebssystem vorgesehenen Mitteln speichert. Das es sich unter Windows hierbei um die Registry handelt, schwante mir erst nach knapp einer Stunde verzweifeltem Suchen. Falls es Euch als UNIX-Nutzern auch mal so geht wie mir: die Voreinstellungen finden sich in der Registry unter:

> HKEY_CURRENT_USER\Software\Javasoft\Prefs\<Package\>

Ja ich weiß, nichts großartiges, aber vielleicht erspare ich damit jemanden das Suchen.

[1]: http://bennat.net
[2]: http://www.jmonkeyengine.com
[3]: http://java.sun.com/javase/6/docs/api/java/util/prefs/Preferences.html
