---
layout: post
title: Wie wende ich die Shannon-Zerlegung an?
author: Martin Thoma
date: 2013-01-17 17:46:27.000000000 +01:00
category: German posts
tags: boolean expression, Digitaltechnik, Boolean algebra
featured_image: 2013/01/george-boole-thumbnail.jpg
---
Die Shannon-Zerlegung ist hilfreich, um die disjunktive bzw. konjunktive Form einer Funktion zu erhalten. Im Folgenden gibt es ein paar Beispiele, wie man das macht:

<h2>Vorgehen</h2>

<ol>
 <li>Man hat eine boolsche Funktion $f(x_1, x_2, \dots, x_n)$ gegeben.</li>
 <li>Entwickeln nach einer Variablen $x_i$:
  <ol>
    <li>$g(x_1, \dots, x_n) := f(x_1, \dots, x_i=1, \dots, x_n)$</li>
    <li>$h(x_1, \dots, x_n) := f(x_1, \dots, x_i=0, \dots, x_n)$</li>
    <li>$f(x_1, x_2, \dots, x_n) = x_i [ g(x_1, \dots, x_n)] \lor \bar x_i [h(x_1, \dots, x_n)]$</li>
  </ol>
  </li>
  <li>Vereinfachen der Funktion</li>
</ol>

<h2>Beispiel 1</h2>
\begin{align}
f(c, b, a) :&= (c \land b) \barwedge a) \Leftrightarrow (b \lor a)\\
\text{Entwickeln nach c:} &= c[((1 \land b) \barwedge a) \Leftrightarrow (b \lor a)] \lor \bar c [((0 \land b) \barwedge a) \Leftrightarrow (b \lor a)]\\
&= c[(b \barwedge a) \Leftrightarrow (b \lor a)] \lor \bar c [(0 \barwedge a) \Leftrightarrow (b \lor a)]\\
&= c[(b \barwedge a) \Leftrightarrow (b \lor a)] \lor \bar c [1 \Leftrightarrow (b \lor a)]\\
&= c((b \barwedge a) \Leftrightarrow (b \lor a)) \lor \bar c (b \lor a)\\
\text{Entwickeln nach b:}&= b \lor \bar b\\
&= b \lor \bar b \\
&= b(c \bar a \lor \bar c) \lor \bar b (ca \lor \bar c a)\\
\text{Entwickeln nach a:} &= a[b(c \bar 1 \lor \bar c) \lor \bar b (c \lor \bar c)] \lor \bar a [b(c \bar 0 \lor \bar c) \lor \bar b (\bar c a)]\\
&=a(b \bar c \lor \bar b) \lor \bar ab\\
&= ab \bar c \lor a \bar b \lor \bar a b\\
&= ab \bar c \lor a \bar b c \lor a \bar b \bar c \lor \bar a b c \lor \bar a b \bar c
\end{align}

Die letzte Darstellung der Funktion $f$ wird Disjunktive Normalform (DNF) genannt. Die vorletzte ist einfach nur eine disjunktion von Konjunktionen.

<h2>Beispiel 2</h2>
\begin{align}
f(c,b,a) &:= ab \lor \bar c\\
\text{Entwickeln nach b:} &= b[a \lor \bar c] \lor \bar b[\bar c]\\
&= b(a \lor \bar c) \lor \bar b \bar c\\
\text{Entwickeln nach a:} &= a[b \lor \bar b \bar c] \lor \bar a [b(\bar c) \lor \bar b \bar c]\\
&= a(b \lor \bar b \bar c) \lor \bar a \bar c\\
&= ab \lor a \bar b \bar c \lor \bar a \bar c\\
&= (abc \lor ab \bar c) \lor a \bar b \bar c \lor (\bar a b \bar c  \lor \bar a \bar b \bar c)
\end{align}

Man muss auch nicht immer Entwicklen, um das Ergebnis zu erhalten. Die Klammern im Ergebnis verdeutlichen, wie man den letzten Schritt durchf&uuml;hrt. Also wie man von einer Disjunktiven Form auf die Disjunktive Normalform kommt.
