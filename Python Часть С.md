# Python. Часть С

## SPOILER C

### Реализовать функцию `roots`, принимающую вектор (список) коэффициентов многочлена (заданы по убыванию степеней) над кольцом ℤ
и возвращающую список, который содержит все целые корни многочлена по неубыванию, с учётом кратности.

Решение:

```python
def divs(n):
    divs = [1, n]
    t = 2
    while t * t <= n:
        if n % t == 0:
            divs.append(t)
            division = n // t
            if division != t:
                divs.append(division)
        t += 1
    return sorted(divs)

def horner(pol, t):
    n = len(pol)
    a = pol[0]
    for i in range(1, n):
        a = a * t + pol[i]
    return a

def roots(pol):
    d = divs(pol[-1])
    ans = []
    for a in d:
        if horner(pol, a) == 0:
            ans.append(a)
        if horner(pol, -a) == 0:
            ans.append(-a)
    return ans
```

### Реализовать менеджер контекстов `open_files(*filenames, mode, encoding)`, открывающий заданные файлы в указанном режиме и кодировке.

Пример использования:

```python
with open_files(‘1.txt’, ‘2.txt’, mode=’r’, encoding=’cp866’) as (f, g):
    print(f.read(), g.read())

```

Решение:

```python
class open_files:

    def __init__(self, *filenames, mode='r', encoding='utf-8'):
        self.filenames = filenames
        self.mode = mode
        self.encoding = encoding
        self.opened_files = []

    def __enter__(self):
        for filename in self.filenames:
            opened_file = open(filename, mode=self.mode, encoding=self.encoding)
            self.opened_files.append(opened_file)
        return tuple(self.opened_files)

    def __exit__(self, exc_type, exc_val, exc_tb):
        for opened_file in self.opened_files:
            opened_file.close()

with open_files('text.txt', 'text2.txt', mode='r', encoding='utf-8') as (f, g):
    print(f.read(), g.read())
```

### Реализовать класс `Rational` рациональных чисел с арифметическими операциями `+`, `-`, `*`, `/`. Арифметика также должна работать с аргументами типа целых чисел.

Пример: `(4 - (Rational(1, 3)*3 + 1)/Rational(2))`

Решение:

```python
import math

class Rational:
    def __init__(self, numerator, denominator=1):
        if denominator == 0:
            raise ZeroDivisionError()
        self._set_values(numerator, denominator)

    def _set_values(self, numerator, denominator):
        self._numerator, self._denominator = self._normalize(numerator,
                                                             denominator)

    @property
    def numerator(self):
        return self._numerator

    @property
    def denominator(self):
        return self._denominator

    def _normalize(self, numerator, denominator):
        gcd = math.gcd(numerator, denominator)
        return numerator // gcd, denominator // gcd

    def __repr__(self):
        return f'Rational({self._numerator}, {self._denominator})'

    def __add__(self, other):
        return Rational(self.numerator * other.denominator
                        + other.numerator * self.denominator,
                        self.denominator * other.denominator)

    def __sub__(self, other):
        return Rational(self.numerator * other.denominator
                        - other.numerator * self.denominator,
                        self.denominator * other.denominator)

    def __mul__(self, other):
        return Rational(self.numerator * other.numerator,
                        self.denominator * other.denominator)

    def __truediv__(self, other):
        return Rational(self.numerator * other.denominator,
                        self.denominator * other.numerator)

    def __radd__(self, integer):
        return Rational(self.numerator + integer * self.denominator,
                        self.denominator)

    def __rsub__(self, integer):
        return Rational(integer * self.denominator - self.numerator,
                        self.denominator)

    def __rmul__(self, integer):
        return Rational(self.numerator * integer, self.denominator)
```

- Настина реализация
    
    ```python
    class Rational:
    
        def __init__(self, *args):
            self.int = args[0]
            if len(args) == 1:
                self.double = 0
            else:
                self.double = args[1]
            self.num = float(self.int + self.double/(10**len(str(self.double))))
    
        def __add__(self, other):
            if isinstance(self, Rational):
                if isinstance(other, Rational):
                    result = self.num + other.num
                else:
                    result = self.num + other
            else:
                if isinstance(other, Rational):
                    result = self + other.num
                else:
                    result = self + other
            return result
    
        def __sub__(self, other):
            if isinstance(self, Rational):
                if isinstance(other, Rational):
                    result = self.num - other.num
                else:
                    result = self.num - other
            else:
                if isinstance(other, Rational):
                    result = self - other.num
                else:
                    result = self - other
            return result
    
        def __mul__(self, other):
            if isinstance(self, Rational):
                if isinstance(other, Rational):
                    result = self.num * other.num
                else:
                    result = self.num * other
            else:
                if isinstance(other, Rational):
                    result = self * other.num
                else:
                    result = self * other
            return result
    
        def __truediv__(self, other):
            if isinstance(self, Rational):
                if isinstance(other, Rational):
                    result = self.num / other.num
                else:
                    result = self.num / other
            else:
                if isinstance(other, Rational):
                    result = self / other.num
                else:
                    result = self / other
            return result
    
        def __rtruediv__(self, other):
            if isinstance(self, Rational):
                if isinstance(other, Rational):
                    result = other.num / self.num
                else:
                    result = other / self.num
            else:
                if isinstance(other, Rational):
                    result = other.num / self
                else:
                    result = other / self
            return result
    
    print(4 - (Rational(1, 3)*3 + 1)/Rational(2))
    ```
    

### Реализовать декоратор `errors_report`, принимающий в качестве параметра функцию `f`. Если функция, к которой применён декоратор, завершилась с исключением `e`, нужно вызвать `f`, передав `e` и пробросить это исключение дальше. Исключения в функции `f` следует игнорировать.

```python
def errors_report(f):
    def decorator(func):
        def wrapper(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except BaseException as e:
                try:
                    f(e)
                except BaseException:
                    pass
                finally:
                    raise e
        return wrapper
    return decorator
```