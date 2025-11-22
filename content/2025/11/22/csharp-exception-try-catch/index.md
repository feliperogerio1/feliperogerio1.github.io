---
title: C# - Exception - try-catch
slug: csharp-exception-try-catch
---

Mais importante do que ter uma aplicação imune a erros, é ter uma aplicação que saiba lidar com erros. Sabemos que é impossível uma aplicação de nível comercial ser 100% protegida de erros, mas sabemos que é possível nos antecipar e preparar o ambiente para que a aplicação consiga responder a esses erros. A seguir vamos explorar algumas práticas para lidar com erros.

## try-catch

Quando falamos do uso do bloco try-catch-finally, precisamos nos atentar a alguns detalhes, pois o uso incorreto pode prejudicar a investigação de erros.
```c# {filename="Program.cs"}
try
{
    int? nullable = null;
    int notNullable = nullable.Value;
}
catch (Exception ex)
{
    // logic for exception handling
    Console.WriteLine($"Exception Message: {ex.Message}");
    Console.WriteLine($"Exception StackTrace: {ex.StackTrace}");
}
```
Este exemplo mostra algumas informações simples que podemos obter a partir de uma exception capturada, como a mensagem do problema e o stacktrace que originou a exception.

## throw

Caso o bloco try-catch-finally esteja envolto por um bloco try-catch-finally de nível superior, é possível utilizar somente a palavra-chave `throw`, que irá fazer com que a exception seja propagada para o bloco de escopo maior, como uma middleware de tratamento de erros por exemplo.
``` c# {filename="Program.cs"}
try
{
    try 
    {
        int? nullable = null;
        int notNullable = nullable.Value;
    }
    catch (Exception ex) 
    {
        throw;
    }
}
catch (Exception ex)
{
    // logic for exception handling
    Console.WriteLine($"Exception Message: {ex.Message}");
    Console.WriteLine($"Exception StackTrace: {ex.StackTrace}");
}
```

## throw new Exception()

Quando for utilizar `throw new Exception()` é necessário entender muito bem o motivo de fazê-lo. Quando ocorre um erro inesperado e queremos interromper o "caminho feliz", é normal lançar uma exception. A seguir está um método que realiza a divisão entre dois números.
``` c# {filename="Program.cs"}
public int Divide(int number1, int number2)
{
    if (number1 == 0)
    {
        throw new Exception($"Value of {nameof(number1)} cannot be zero.");    
    }
    if (number2 == 0)
    {
        throw new Exception($"Value of {nameof(number2)} cannot be zero");
    }
    
    return number1 / number2;
}
```

Se passarmos um dos argumentos com o valor 0, irá lançar uma exception.
``` c# {filename="Program.cs"}
try 
{
    Console.WriteLine(Divide(number1: 5, number2: 0));
}
catch (Exception ex)
{
    // logic for exception handling
    Console.WriteLine($"Exception Message: {ex.Message}");
    Console.WriteLine($"Exception StackTrace: {ex.StackTrace}");
}
```

Agora caso quisermos propagar essa exception para um escopo superior, temos duas formas: uso de somente a palavra-chave `throw` ou `throw new Exception()` informando `message` e `innerException` como argumentos.

Uso de `throw`:
``` c# {filename="Program.cs"}
try 
{
    try 
    {
        Console.WriteLine(Divide(number1: 5, number2: 0));
    }
    catch (Exception ex)
    {
        throw;
    }
}
catch (Exception ex)
{
    // logic for exception handling
    Console.WriteLine($"Exception Message: {ex.Message}");
    Console.WriteLine($"Exception StackTrace: {ex.StackTrace}");
}
```

Uso de `throw new Exception`:
``` c# {filename="Program.cs"}
try 
{
    try 
    {
        Console.WriteLine(Divide(number1: 5, number2: 0));
    }
    catch (Exception ex)
    {
        throw new Exception(message: $"Something went wrong! {ex.Message}", innerException: ex);
    }
}
catch (Exception ex)
{
    // logic for exception handling
    Console.WriteLine($"Exception Message: {ex.Message}");
    Console.WriteLine($"Exception StackTrace: {ex.StackTrace}");
    Console.WriteLine($"InnerException: {ex.InnerException}");
}
```

Normalmente o uso de `throw new Exception()` para propagar exceptions é intessante quando precisamos manipular o valor de `message` da exception. Quando estamos nessa situação, precisamos passar a exception de escopo inferior como `innerException` da nova exception para não perdermos as informações da exception original, como o `stackTrace`.