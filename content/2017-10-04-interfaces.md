---
title: "Про интерфейсы"
date: 2017-10-04T17:39:21+05:00
url: /2017/10/04/interfaces/
author: sonicgd
type: post
categories:
  - 'PHP vs C#'
---

Реализация интефрейсов в C# тоже сделана не так прямолинейно как в PHP.

Типичный интерфейс:
```csharp
interface IFoo
{
    void Foo();
}
```
Обычная реализация выглядит так:
```csharp
class Foo : IFoo
{
    public void Foo()
    {
      Console.WriteLine("Foo");
    }
}
```
И мы можем использовать класс вот так:
```csharp
var foo = new Foo();
foo.Foo(); //работает
```
Совсем как PHP. Или нет? 

<!--more-->
В примере мы создаём в классе публичный метод, который "неявно" реализует интерфейс. Но мы можем "явно" реализовать интерфейс вот таким образом
```csharp
class Foo2 : IFoo
{
    void IFoo.Foo()
    {
        Console.WriteLine("Foo");
    }
}
```
В этом случае, если мы попытаемся напрямую вызвать метод `Foo`, то мы словим ошибку, точнее код не скомпилируется, потому что у класса нет такого публичного метода
```csharp
var foo2 = new Foo2();
foo2.Foo(); //ошибка
```
Однако, если мы приведём объект к интерфейсу, то мы сможем вызвать у него этот метод
```csharp
var foo2 = new Foo2();
((IFoo) foo2).Foo(); //Foo
```
Но и это ещё не всё :) Мы можем объединить обе эти реализации
```csharp
class AnotherFoo : IFoo
{
    public void Foo()
    {
        Console.WriteLine("Bar");
    }
    
    void IFoo.Foo()
    {
        Console.WriteLine("Foo");
    }
}
```
И теперь написать такой код:
```csharp
var anotherFoo = new AnotherFoo();
anotherFoo.Foo(); //Bar - потому что вызовется публичный метод класса
((IFoo) anotherFoo).Foo(); //Foo - потому что вызовется метод, реализующий интерфейс
```
Ну и, конечно же, это работает с несколькими интерфейсами
```csharp
interface IBaz
{
    void Foo();
}

class Baz : IFoo, IBaz
{
    public void Foo()
    {
        Console.WriteLine("Bar");
    }
    
    void IFoo.Foo()
    {
        Console.WriteLine("Foo");
    }
    
    void IBaz.Foo()
    {
        Console.WriteLine("Baz");
    }
}
```
И использовать можно так:
```csharp
var baz = new Baz();
baz.Foo(); //Bar
((IFoo) baz).Foo(); //Foo
((IBaz) baz).Foo(); //Baz
```

