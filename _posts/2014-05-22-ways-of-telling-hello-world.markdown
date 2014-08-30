---
layout: post
title: "Ways of Telling Hello, world"
date: 2014-05-22 22:59:00
author: hbc
categories: fun programming
---

Every programmer loves [Hello, world](http://en.wikipedia.org/wiki/Hello_world),
and I know some ways to tell this incantation.


C:

```c
#include <stdio.h>

int main()
{
    printf("Hello, world");

    return 0;
}
```

Python:

```python
print('Hello, world')
```


Ruby:

```ruby
puts "Hello, world"
```


JavaScript:

```javascript
console.log('Hello, world');
```


PHP:

```php
<?php
echo 'Hello, world';
```


C++:

```c++
#include <iostream>

using namespace std;

int main()
{
    cout << "Hello, world" << endl;

    return 0;
}
```


Java:

```java
public class HelloWorld {
    public static void main(String[] args) {
      System.out.println("Hello, world");
    }
}
```


C#:

```c#
using System;

class HelloWorld
{
    static void Main()
    {
        Console.WriteLine("Hello, world");
    }
}
```


Golang:

```go
package main

import "fmt"

func main() {
        fmt.Println("Hello, world")
}
```


HTML:

```html
<h1>Hello, world</h1>
```


Shell:

```bash
$ echo "Hello, world"
```


Please enjoy your programming adventure!
