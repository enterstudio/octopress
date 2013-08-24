--- 
layout: post
title: Maven Dependency Ranges
categories: 
- software-entwicklung
- java
status: publish
type: post
published: true
---
Meine Hassliebe zu Maven begleitet mich nun schon eine ganze Weile, aber ich halte alle anderen Build-Systeme die ich kenne (na gut, unter Java ist es nur Ant) für noch viel schlimmer.

Dieser Unwille sich mit Maven näher als unbedingt nötig auseinander zu setzen hat auch dazu geführt, dass mir bis jetzt nie klar wurde, dass es möglich ist, für Abhängigkeiten des Projektes Versionsbereiche anzugeben, also: "Nimm die aktuellste Version größer X aber nicht aktueller als Y." Das geht recht einfach, nämlich folgendermaßen:

{% codeblock lang:xml %}
 <dependency>
     <groupId>junit</groupId>
     <artifactId>junit</artifactId>
     <version>[3.8,4.0)</version>
     <scope>test</scope>
 </dependency>
{% endcodeblock %}

Würde die aktuelleste JUnit3 Release nutzen, jedoch keine aktuellere Release. Das Beispiel ist natürlich aus der [Dokumentation][1] von Maven geklaut. 

{% codeblock lang:xml %}
 <version>[2.0,)</version>
{% endcodeblock %}

würde dementsprechend die aktuellste Version größer 2.0 verwenden. Weitere Informationen zu den Dependency Ranges stehen in der [Dokumentation.][1] 

[1]: http://www.sonatype.com/books/mvnref-book/reference/pom-relationships-sect-version-ranges.html
