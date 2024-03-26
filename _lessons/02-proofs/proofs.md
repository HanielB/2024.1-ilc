---
layout: page
title: Demonstrações
---

# Demonstrações
{: .no_toc .mb-2 }

## Leituras

- [Notas de aula sobre demonstrações e regras de inferências]({{ site.baseurl }}{% link _lessons/02-proofs/04-proofs.pdf %}).
- Do livro texto:
  - Regras de inferência: capítulo 1.5

- [Notas de aula sobre métodos de demonstração]({{ site.baseurl }}{% link _lessons/02-proofs/06-proofs.pdf %}).
- Do livro texto:
  - Introdução demonstrações: capítulo 1.6
  - Métodos de demonstração: capítulo 1.7

## Automatizando busca de demonstrações

O solucionador SMT cvc5 pode ser usado para demonstrações utilizando a dualidade entre *validade* (isto é, uma fórmula ser *sempre* verdadeira, ou seja uma *tautologia*) entre e *satisfatibilidade* (isto é, uma fórmula *poder* ser verdadeira). Uma fórmula `φ` é uma *tautologia* se e somente se `¬φ` é *insatisfatível*.

O exemplo então de determinar se a partir das premissas `p → q`, `r v s`, `r → t`, `¬q`, `u → v`, `s → p` podemos *sempre* concluir `t`, ou seja, se podemos demonstrar que 
```
((p → q) ∧ (r v s) ∧ (r → t) ∧ ¬q ∧ (u → v) ∧ (s → p)) → t
```
é uma tautoligia, pode ser reduzida a demonstrar que sua negação, ou seja, 
```
((p → q) ∧ (r v s) ∧ (r → t) ∧ ¬q ∧ (u → v) ∧ (s → p)) ∧ ¬t
```
é insatisfatível.

O programa abaixo utiliza o cvc5 para fazer esse teste:

```python
from cvc5.pythonic import *

if __name__ == '__main__':
    p, q, r, s, t, u, v = Bools("p q r s t u v")

    solver = Solver()

    # premissas
    solver.add(Or(Not(p), q))
    solver.add(Or(r, s))
    solver.add(Or(Not(r), t))
    solver.add(Not(q))
    solver.add(Or(Not(u), v))
    solver.add(Or(Not(s), p))

    # conclusão (negada)
    solver.add(Not(t))

    print(solver.check())
```

Como o resultado de rodar esse programa é `unsat`, temos que a fórmula correspondendo à negação do que queremos demonstrar é insatisfatível. Logo, o cvc5 nos diz que a fórmula 
```
((p → q) ∧ (r v s) ∧ (r → t) ∧ ¬q ∧ (u → v) ∧ (s → p)) → t
```
é uma tautologia.
