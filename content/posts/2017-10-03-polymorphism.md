---
title: "Про полиморфизм"
date: 2017-10-03T15:30:15+05:00
categories:
  - 'PHP vs C#'
---

После PHP в C# не сразу очевидно как работает переопределение методов при наследовании. В PHP всё просто:

```php
<?php

class Foo {
    public function test()
    {
        echo 'Foo';
    }
    
    public function proxyTest()
    {
        $this->test();
    }
}

class Bar extends Foo {
    public function test()
    {
        echo 'Bar';
    }
}

$foo = new Foo();
$foo->test(); //Foo
$foo->proxyTest(); //Foo

$bar = new Bar();
$bar->test(); //Bar
$bar->proxyTest(); //Bar
```

Переопределённый метод заменяет реализацию базового класса целиком и полностью. В C# всё немного веселее.
<!--more-->


Создадим такие же классы `Foo` и `Bar`:

```csharp
class Foo
{
    public void Test()
    {
        Console.WriteLine("Foo");
    }

    public void ProxyTest()
    {
        Test();
    }
}

class Bar : Foo
{
    public void Test()
    {
        Console.WriteLine("Bar");
    }
}
```

И точно так же вызовем методы:

```csharp
class Program
{
    static void Main(string[] args)
    {
        var foo = new Foo();
        foo.Test(); //Foo
        foo.ProxyTest(); //Foo
        
        var bar = new Bar();
        bar.Test(); //Bar
        bar.ProxyTest(); //Foo <- ???
    }
}
```

Не так как в PHP. Более того, попутно мы ещё и получим предупреждение от компилятора:

```
warning CS0108: 'Bar.Test()' hides inherited member 'Foo.Test()'. Use the new keyword if hiding was intended.
```

Переопределив метод таким образом, мы на самом деле просто "скрыли" его в классе-наследнике, однако методы базового класса по прежнему будут видеть и обращаться именно к базовой реализации. Новый же метод будет доступен только для членов класса `Bar` и при вызове у объекта класса `Bar`. То есть, если мы приведём `Bar` к `Foo`, то вызовется именно базовый метод:

```csharp
class Program
{
    static void Main(string[] args)
    {
        var bar = new Bar();
        bar.Test();
        bar.ProxyTest();
        ((Foo) bar).Test(); //Foo
    }
}
```

Компилятор подсказывает нам, что если такое поведение было задумано, то стоит явно об этом сообщить, добавив к методу ключевое слово `new`.

```csharp
class Bar : Foo
{
    public new void Test()
    {
        Console.WriteLine("Bar");
    }
}
```

Всё продолжит работать как работало, просто исчезнет предупреждение. Однако, что если мы хотим именно переопределить метод? Ну, как в старом добром PHP? Здесь нам помогут ключевые слова `virtual` и `override`.

Чтобы сделать метод переопределяемым — надо пометить его как `virtual`:

```csharp
class Foo
{
    public virtual void Test()
    {
        Console.WriteLine("Foo");
    }

    public void ProxyTest()
    {
        Test();
    }
}
```

Чтобы переопределить метод — надо пометить новый метод как `override`:

```csharp
class Bar : Foo
{
    public override void Test()
    {
        Console.WriteLine("Bar");
    }
}
```

Запустим программу ещё раз:

```csharp
class Program
{
    static void Main(string[] args)
    {
        var foo = new Foo();
        foo.Test(); //Foo
        foo.ProxyTest(); //Foo
        
        var bar = new Bar();
        bar.Test(); //Bar
        bar.ProxyTest(); //Bar
        ((Foo) bar).Test(); //Bar
        ((Foo) bar).ProxyTest(); //Bar
    }
}
```

Вот, теперь всё именно так, как мы и хотели — новый метод полностью подменил старый. Создав объект класса `Bar` мы всегда будем вызывать новую реализацию.