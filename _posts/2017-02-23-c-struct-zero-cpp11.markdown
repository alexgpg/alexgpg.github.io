---
layout: post
title:  "Зануляем Plain C структуры в C++ >= C++11"
date:   2017-02-23 00:00:01
categories: posts
---

Некоторые структуры Plain C требуют зануления перед использованием и можно
видеть вот такой код, который делает это. Для примера будет использоваться
```struct sockaddr_in```.

{% highlight c %}
struct sockaddr_in addr;
memset(&addr, 0, sizeof(addr));
{% endhighlight %}

Вместо ```memset``` может быть ```bzero```.

Начиная с С++11 тоже самое можно записать компактнее, используя синтаксис
с фигурными скобками(braced-init-list):

{% highlight c++ %}
sockaddr_in addr{};
{% endhighlight %}

Такой объявление занулит ```addr```.

Для [POD типов](http://en.cppreference.com/w/cpp/concept/PODType), к котором
относятся структруры Plain C будет применена  value-initializtion, которая
влечет zero-initializtion, что означает заполнение структуры нулевыми значениями.

Работает и с ```new```

{% highlight c++ %}
sockaddr_in *addr = new sockaddr_in{};
{% endhighlight %}

и в constructor initializer lists

{% highlight c++ %}
class Connection {
 public:
  Connection() : addr_{} { }

  sockaddr_in addr_;
};
{% endhighlight %}

и вообще везде, где работает value initialization.

### Проверка

Убедиться, что это работает именно так, можно используя
[онлайн-компилятор godbolt](https://godbolt.org/).

Пример:

{% raw %}
<iframe width="800px" height="200px" src="https://godbolt.org/e#compiler:g520,filters:'colouriseAsm,compileOnChange,labels,directives,commentOnly',options:'-std%3Dc%2B%2B11+-O3',source:'%23include+%3Cnetinet/in.h%3E%0A%23include+%3Ccstring%3E%0A%0Astatic+void+Escape(void+*p)+%7B%0A++asm+volatile(%22%22+:+:+%22g%22(p)+:+%22memory%22)%3B%0A%7D%0A%0Aint+main()+%7B%0A+sockaddr_in+addr%7B%7D%3B%0A+Escape(%26addr)%3B++%0A+return+0%3B%0A%7D'"></iframe>
{% endraw %}

Видно, что комплиятор вставляет инструкции для зануления структуры.

Функция Escape - хитрый трюк, чтобы компилятор не мог понять что ```addr```
не используется и не выкинул инициализацию при оптимизации.

### Заключение

Это просто небольшое удобство, которое позволяет совсем чуть-чуть сократить код
при работе с Plain C структурами, используя возможности новых стандартов.

Далее идет нудный разбор почему это должно так работать по стандарту.

### Объяснение и стандарт

Для разбора будет использоваться
[черновик стандарта C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf).

Синтаксис с фигурными скобками ```{}``` влечет list-initialization.

$ 8.5.4, List-initialization, пункт 3, страница 199

~~~
List-initialization of an object or reference of type T is defined as follows:
 — If the initializer list has no elements and T is a class type with a default
constructor, the object is value-initialized.
~~~

Т.е. для value-initialization нужен пустой список и конструктор по умолчанию.
Пустой список есть из синтаксиса.
Конструктор по умолчанию также есть, его создает компилятор. Это можно узнать из

$ 12.1 Constructors, пункт 6, страница 243

~~~
A default constructor that is defaulted and not defined as deleted is implicitly
defined when it is odrused (3.2) to create an object of its class type (1.8)
or when it is explicitly defaulted after its first declaration.
The implicitly-defined default constructor performs the set of initializations
of the class that would be performed by a user-written default constructor
for that class with no ctor-initializer (12.6.2) and an empty
compound-statement.
~~~

Из пункта ниже можно понять, что компилятор создаст тривиальный конструктор
т.к. наша Plain C структура очень простая.

$ 12.1 Constructors, пункт 5, страница 243

~~~
A default constructor is trivial if it is not user-provided and if:
 — its class has no virtual functions (10.3) and no virtual base classes (10.1),
   and
 — no non-static data member of its class has a brace-or-equal-initializer, and
 — all the direct base classes of its class have trivial default constructors,
   and
 — for all the non-static data members of its class that are of class type
   (or array thereof), each such class has a trivial default constructor.
~~~

Исходя из этого, видим что для структуры будет использована
value-initialization. Переходим к тому, что в результате такой инициализации
произойдет.

$ 8.5 Initializers, пункт 7, страница 191

~~~
if T is a (possibly cv-qualified) non-union class type without a user-provided
constructor, then the object is zero-initialized and, if T’s implicitly-declared
default constructor is non-trivial, that constructor is called.
~~~

Т.о. для структуры используется zero-initialized, а т.к. конструктор тривиальный
то больше ничего не происхоит, что означает что структура будет заполнена нулями,
что нам и требовалось.

И еще, в стандарте используется class, структура тоже class. Об этом можно
узнать из $ 9, Classes, пункт 1, страница 204.

Задействованные пункты стандарта

* $ 12.1 Constructors, пункт 5 и 6, страница 243
* $ 8.5.4 List-initialization, пункт 3, страница 199
* $ 8.5 Initializers, пункт 7, страница 191
* $ 9 Classes, пункт 1, страница 204

### Ссылки
  * [value initialization](http://en.cppreference.com/w/cpp/language/value_initialization)
  * [Default constructors](http://en.cppreference.com/w/cpp/language/default_constructor)
  * [Хорошая статья: Initialization in C++ is bonkers](http://blog.tartanllama.xyz/c++/2017/01/20/initialization-is-bonkers/)
  * [POD(Plain Old Data) type](http://en.cppreference.com/w/cpp/concept/PODType)
  * [godbolt: пример использования](https://godbolt.org/g/lM0bA3)
  * [Черновик(draft) стандарта C++11](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf)
