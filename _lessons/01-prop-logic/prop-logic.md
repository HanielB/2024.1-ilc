---
layout: page
title: Lógica proposicional
---

# Lógica proposicional
{: .no_toc .mb-2 }

- TOC
{:toc}

## Leituras

- Notas de aula:
  - [Introdução a Lógica Proposicional]({{ site.baseurl }}{% link _lessons/01-prop-logic/01-prop-logic-intro.pdf %})
  - [Aplicações e equivalências]({{ site.baseurl }}{% link _lessons/01-prop-logic/02-prop-logic-apps_equiv.pdf %})
  - [Satisfatibilidade]({{ site.baseurl }}{% link _lessons/01-prop-logic/03-prop-logic-sat.pdf %})
- Do livro texto:
  - Conectivos lógicos: capítulo 1.1
  - Aplicações, equivalências: capítulo 1.2

## Resolução de problemas via SAT

Podemos resolver problemas SAT automaticamente usando ferramentas de *automatização de raciocínio*. Um exemplo é o [cvc5](https://cvc5.github.io/), um solucionador SMT (de *satisfatibilidade módulo teorias*; falaremos um pouco mais sobre SMT em futuras aulas).

Como vimos em aula, podemos usar a [API](https://pt.wikipedia.org/wiki/Interface_de_programa%C3%A7%C3%A3o_de_aplica%C3%A7%C3%B5es) do cvc5 em Python para escrever programas nessa linguagem que resolvem problemas que podemos representar via SAT.

Para instalar o `cvc5` e usar sua API em Python, basta instalar o cvc5 com, por exemplo:

``` shell
pip install cvc5
```

Mais instruções sobre instalação estão disponíveis [aqui](https://cvc5.github.io/docs/cvc5-1.0.7/api/python/python.html). E a documentação das operações disponíveis na API estão [aqui](https://cvc5.github.io/docs/cvc5-1.0.7/api/python/pythonic/pythonic.html).

### Determinando a satisfatibilidade de uma fórmula proposicional

O programa abaixo, ao ser executado, apresnta todas as soluções para a fórmula

```
(p v q v r) ∧ (¬p v ¬q v ¬r)
```

```python
from cvc5.pythonic import *

if __name__ == '__main__':
    p, q, r = Bools("p q r")
    s = Solver()

    # p v q v r
    s.add(Or(p, q, r))
    # ¬p v ¬q v ¬r
    s.add(Or(Not(p), Not(q), Not(r)))

    count = 0
    while (s.check() == sat):
        m = s.model()
        print("Solution {}".format(count))
        print("p: ", m[p])
        print("q: ", m[q])
        print("r: ", m[r])
        s.add(Or(p != m[p], q != m[q], r != m[r]))
        print("============")
        count += 1
```

### Determinando a equivalência de duas fórmulas proposicionais

O programa abaixo, ao ser executado, testa a equivalência entre as fórmulas `¬(p ∨ (¬p ∧ q))` e `¬p ∧ ¬q`. Se elas são equivalentes então a bi-implicação entre elas deve ser uma tautologia. O que significa que a *negação* da bi-implicação deve ser uma contradição, ou seja, deve ser *insatisfatível*. Na linguagem do `cvc5`, a bi-implicação é representada com `==` e sua negação com `!=`.

Então, se o resultado for `unsat`, as fórmulas são equivalentes. Caso contrário existe uma maneira de dar valores às suas variáveis que as faz ter valores de verdade diferentes.


```python
from cvc5.pythonic import *

if __name__ == '__main__':
    p, q = Bools("p q")
    s = Solver()

    # Testar equivalencia de ¬(p ∨ (¬p ∧ q)) e ¬p ∧ ¬q
    s.add(
        Not(Or(p, (And(Not(p), q)))) != And(Not(p), q))

    print(s.check())
```

### Resolvendo o problema das n-rainhas

Segundo a codificação dada nas [notas de aula de satisfatibilidade]({{ site.baseurl }}{% link _lessons/01-prop-logic/03-prop-logic-sat.pdf %}), podemos escrever o programa abaixo que dá todas as soluções para o problema das `n` rainhas, para um dado `n`.

```python
from cvc5.pythonic import *

# Definindo o número de rainhas
n = 8

if __name__ == '__main__':
    # Criamos n+1 posições para que possamos contar de 1 até n (em vez de 0
    # até n-1, como seria o padrão)
    board = [[None for i in range(n+1)] for i in range(n+1)]
    # Criamos uma variável para cada posição no tabuleiro
    for i in range(1, n+1):
        for j in range(1, n+1):
            board[i][j] = Bool("p{}{}".format(i, j))

    s = Solver()
    # Q1: há pelo menos uma rainha por linha
    for i in range(1, n+1):
        row = []
        for j in range(1, n+1):
            row += [board[i][j]]
        s.add(Or(row))

    # Q2: há no máximo uma rainha por linha
    for i in range(1, n+1):
        for j in range(1, n):
            for k in range(j+1, n+1):
                s.add(Implies(board[i][j], Not(board[i][k])))

    # Q3: há no máximo uma rainha por coluna
    for j in range(1, n+1):
        for i in range(1, n):
            for k in range(i+1, n+1):
                s.add(Implies(board[i][j], Not(board[k][j])))

    # Q4: não há rainhas na mesma diagonal (parte 1)
    for i in range(2, n+1):
        for j in range(1, n):
            for k in range(1, min(i-1, n-j) + 1):
                s.add(Implies(board[i][j], Not(board[i-k][k+j])))

    # Q5: não há rainhas na mesma diagonal (parte 2)
    for i in range(1, n):
        for j in range(1, n):
            for k in range(1, min(n-i, n-j) + 1):
                s.add(Implies(board[i][j], Not(board[i+k][j+k])))

    count = 0
    while (s.check() == sat):
        m = s.model()
        values = []
        count += 1
        print("Solution {}\n----------".format(count))
        for i in range(1, n+1):
            string = ""
            for j in range(1, n+1):
                string += "{}{}".format("Q" if m[board[i][j]] else "_", ", " if j < n else "")
                values += [board[i][j] != m[board[i][j]]]
            print(string)
        print("===============================================")
        # block current solution
        s.add(Or(values))
```
