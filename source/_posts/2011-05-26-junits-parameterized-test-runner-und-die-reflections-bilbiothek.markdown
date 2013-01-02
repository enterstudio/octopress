--- 
layout: post
title: Junits Parameterized Test-Runner und die Reflections-Bilbiothek
categories: 
- java
- software-entwicklung
status: publish
type: post
published: true
---

Im Rahmen eines Projektes stieß ich auf ein ein interessantes Problem: Alle meine mit [Hibernate][1] persistierten Entity-Klassen sollen ein Interface implementieren, das folgendermaßen aussieht:

{% codeblock lang:java %}
public interface EntityIf extends Serializable {
    Long getId();

    Date getCreated();

    UUID getUUID();

    @Override
    String toString();
}
{% endcodeblock %}

Um das Problem noch etwas interessanter zu gestalten, wurde  eine abstrakte Klasse geschrieben, die
eben dieses Interface implementiert und von dem dann alle Wald- und Wiesen-Entities ableiten können. 
Auf diese Weise soll das "Copy 'n' Waste threshold" reduziert werden.

Um sicher zu stellen, dass die Entities im System dieses Interface nicht nur implementieren, sondern 
sich auch so verhalten, wie es ursprünglich gedacht war, sollen alle Entities mit einem Unit-Test, der 
die im Interface definierten Methoden testet,  eben darauf geprüft werden. Mit [JUnit][2] ist dieser 
Test schnell geschrieben: JUnit 4 bietet mit dem ''[Parameterized Test Runner][3]'' die Möglichkeit, 
diese einmal geschriebene Unit-Tests gegen zur Laufzeit generierte Testdaten auszuführen. Dazu muss 
eine Testklasse folgende Bedingungen erfüllen:

* Die Test-Klasse muss mit dem *Parameterized Test-Runner* ausgeführt werden. Dies wird erreicht, indem man die Testklasse mit `@RunWith(Parameterized.class)` annotiert.
* Eine statische Methode, welche die Test-Daten erzeugt.
* Ein einziger Konstruktor, der als Parameter die erzeugten Test-Daten akzeptiert und diese für die Tests vorhält.
* Natürlich mindestens eine Test-Methode (annotiert mit `@Test`)

Die statische Methode, welche die Test-Daten erzeugt, muss mit `@Parameters` annotiert werden und eine 
`Collection` von Arrays zurück geben. Die Anzahl derElemente eines Arrays muss der Anzahl an Parametern 
des Konstruktors entsprechen, denn die Arrays werden genutzt um zur Laufzeit der Klasse die Testobjekte 
zu erzeugen.

[JUnit 4][2] stellt also eine Möglichkeit zur Verfügung, mit relativ einfachen Mitteln Tests zur 
Laufzeit dynamisch zu erzeugen. Es fehlt nur noch eine Möglichkeit, zur Laufzeit alle mein Interface 
implementierenden Klassen zu identifizieren. Es stellte sich heraus, dass das schwieriger zu 
realisieren war, als erwartet. Die JVM bietet nämlich von Hause aus keine einfache Methode, um alle zur 
Laufzeit im Classloader zur Verfügung stehenden Implementierungen einer Klasse zu ermitteln. Auch das 
fantastische Buch "[Java Reflection in Action][4]" zeigte keinen gangbaren Weg auf.

Zur Hilfe kam die  Bibliothek [Reflections][5], die unter der LGPL bei [Google-Code][6] bezogen werden 
kann. Reflections scannt den Classpath des einbindenden Projekts, indiziert die Metadaten der Elemente 
des Classpaths und stellt diese für Abfragen bereit. Die oben beschriebene Suche nach allen (indirekt, 
also auch in zweiter Generation) ein Interface implementierenden Klassen als Daten-Provider für meinen 
Unit-Test mit den oben beschriebenen ''[Parameterized Test Runner][3]'' sieht mit dem Einsatz der
 Bibliothek wie folgt aus:

{% codeblock lang:java %}
    @Parameters
    public static Collection dataParameters() throws InstantiationException, IllegalAccessException {
       Reflections reflections = new Reflections(new ConfigurationBuilder()
                .setUrls(
                    ClasspathHelper.getUrlsForPackagePrefix("org.balumba.project")
                 )
                .setScanners(new SubTypesScanner()));
        Set<Class<? extends EntityIf>> subTypes = reflections.getSubTypesOf(EntityIf.class);
        List<Object[]> result = new ArrayList<Object[]>();
        for (Class cls : subTypes) {
            if (Modifier.isAbstract(cls.getModifiers())) {
                continue;
            }
            Object[] entry = new Object[]{cls.newInstance()};
            result.add(entry);
        }
        System.out.println();
        return result;
    }
{% endcodeblock %}

Von Bedeutung sind hier besonders die Zeilen drei bis acht, die mittels des `ConfigurationBuilder`
die Reflections Bibliothek konfiguriert. Der verwendete [SubTypeScanner][7] scannt den Classpath nach 
Superklassen und Interfaces, und stellt diese Informationen für spätere Abfragen bereit. Die Zeile 
acht liefert dann ein [Set][8] aller Sub-Typen des EntityIf-Interfaces. Zeile 12 stellt sicher, dass 
keine abstrakte Klasse in den Test-Daten landet, denn die kann ja vom Test dann nicht instantiiert 
werden. Hier wird die normale Reflections-Api von Java genutzt, [Modifier][9] ist eine Klasse, die Zugriff auf die Modifier einer Klasse liefert.

Mit Hilfe der abgebildeten `dataParameters()`-Methode ist es nun möglich mit den üblichen Test-Methoden 
sicher zu stellen, dass sich alle ein Interface implementierenden Klassen so verhalten, wie es vom Rest 
der Applikation benötigt. wird.

Die Reflections-Bibliothek bietet auch ein Maven-Plugin, mit der die Metadaten zur Build-Zeit ermittelt 
werden können, um dann zur Laufzeit auf das Scannen verzichten zu können, allerdings habe ich das 
bisher noch nicht probiert. Einen kleinen Wermutstropfen scheint es noch zu geben: Die Bibliothek hat 
schon seit geraumer Zeit keine Aktualisierung mehr erfahren, es ist nicht klar, ob es mit diesem 
Projekt noch lang weiter geht. Das wird aber die Zeit zeigen.

[1]: http://www.hibernate.org/
[2]: http://www.junit.org/
[3]: http://blogs.oracle.com/jacobc/entry/parameterized_unit_tests_with_junit
[4]: http://www.manning.com/forman/
[5]: http://code.google.com/p/reflections/
[6]: http://code.google.com/
[7]: http://reflections.googlecode.com/svn/trunk/reflections/javadoc/apidocs/org/reflections/scanners/SubTypesScanner.html
[8]: http://download.oracle.com/javase/6/docs/api/java/util/Set.html
[9]: http://download.oracle.com/javase/6/docs/api/java/lang/reflect/Modifier.html
