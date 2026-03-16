---
layout: post
title: "Dziedziny, pochodne i macierze"
date: 2026-03-16 13:40:00 +0100
description: 
img: zestaw2.png
---

# Zadanie 1 — Dziedzina funkcji (podpunkt i)

Rozważamy funkcję:

<div markdown="0">
$$
f(x,y) = \sqrt{x^2 - y^2}
$$
</div>

## Warunek istnienia pierwiastka

Aby pierwiastek był określony, wyrażenie pod nim musi być nieujemne:

<div markdown="0">
$$
x^2 - y^2 \ge 0
$$
</div>

## Przekształcenie warunku

<div markdown="0">
$$
x^2 \ge y^2
$$
</div>

co jest równoważne:

<div markdown="0">
$$
|x| \ge |y|
$$
</div>

## Dziedzina funkcji

<div markdown="0">
$$
D = \{(x,y) \in \mathbb{R}^2 : x^2 - y^2 \ge 0 \}
$$
</div>

czyli wszystkie punkty spełniające:

<div markdown="0">
$$
-|x| \le y \le |x|
$$
</div>

## Interpretacja geometryczna

Granice dziedziny wyznaczają proste:

<div markdown="0">
$$
y = x
$$
</div>

<div markdown="0">
$$
y = -x
$$
</div>

Dziedzina to obszar pomiędzy tymi prostymi, w którym spełniony jest warunek

<div markdown="0">
$$
|x| \ge |y|
$$
</div>

Graficznie tworzy to dwa symetryczne obszary otwierające się w lewo i w prawo względem osi \(x\).

![Wykres dziedziny](/assets/img/dziedzina_wykres.png)

## Podsumowanie

Dziedziną funkcji są wszystkie punkty płaszczyzny spełniające warunek:

<div markdown="0">
$$
x^2 - y^2 \ge 0
$$
</div>

czyli punkty leżące pomiędzy prostymi \(y=x\) oraz \(y=-x\) (łącznie z tymi prostymi).
