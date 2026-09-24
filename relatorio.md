# E1 - Analisador léxico - Relatório

Aluno: Pablo Caldeira

Todos os níveis passaram no `testar.py` (0 a 7). Abaixo estão os autômatos que eu escrevi no `afds.py`. Os desenhos estão em texto: `-->` marca o estado inicial, `(( ))` é estado final e `↺` é laço no próprio estado. O estado de erro não aparece porque o `afd()` acrescenta sozinho.

---

## Nível 1 - Inteiro

```
          dígito
--> (n0) --------> ((n1)) ↺ dígito
```

- **n0**: ainda não li nenhum dígito.
- **n1**: já li pelo menos um dígito, então o que foi lido até aqui é um inteiro.

**Estados: 2.** O inteiro é "um ou mais dígitos", então a única coisa que o autômato precisa saber é se já viu um dígito ou não. O laço em n1 cuida do "mais dígitos". No caso `9x` o `x` vai pro erro, o casamento mais longo para no `9` e o `x` vira ID depois.

---

## Nível 2 - Atribuição e ponto e vírgula

```
          =                          ;
--> (a0) ----> ((a1))      --> (p0) ----> ((p1))
```

- **a0 / p0**: não li nada ainda.
- **a1 / p1**: li o `=` (ou o `;`), token completo.

**Estados: 2 em cada.** São tokens de um caractere só, então basta "antes" e "depois" do caractere. Não coloquei laço nenhum, porque se tivesse laço em a1 o `==` virava um token só, e o teste espera dois `ATRIB`.

---

## Nível 3 e 5 - Palavras reservadas

Não tem autômato novo. As palavras `inteiro`, `real` e `logico` foram pro dicionário `PALAVRAS` como `TIPO`, e `verdadeiro` e `falso` como `LOGICO`. Elas são reconhecidas pelo próprio `afd_id` e depois o analisador troca a categoria se o lexema estiver na tabela. Por isso `inteirox` e `falsox` continuam sendo ID: o identificador casa a palavra inteira e ela não está na tabela.

---

## Nível 4 - Real

```
          dígito          .          dígito
--> (r0) --------> (r1) -----> (r2) --------> ((r3))
                    ↺ dígito                   ↺ dígito
```

- **r0**: não li nada.
- **r1**: li a parte inteira (um ou mais dígitos), ainda não vi o ponto.
- **r2**: li a parte inteira e o ponto, mas nenhum dígito depois dele.
- **r3**: li parte inteira, ponto e pelo menos um dígito depois. Real completo.

**Estados: 4.** Precisa separar quatro situações diferentes: antes de tudo, dentro da parte inteira, logo depois do ponto e dentro da parte decimal. Só o r3 é final. O r1 não pode ser final porque aí o `42` seria real e não inteiro (e quem reconhece o inteiro é o outro autômato). O r2 não pode ser final porque aí o `12.` passava como real.

---

## Nível 6 - Comentário de linha

```
          /          /
--> (c0) ----> (c1) ----> ((c2)) ↺ qualquer coisa menos \n
```

- **c0**: não li nada.
- **c1**: li uma barra só. Ainda não é comentário.
- **c2**: li as duas barras, estou dentro do comentário até a quebra de linha.

**Estados: 3.** Um para cada barra e um para o corpo do comentário. O c1 não é final, então uma barra sozinha dá erro léxico, que é o que o teste pede. No c2 o laço usa `SIGMA - {'\n'}` pra parar antes da quebra de linha; a quebra fica pro `afd_branco`.

---

## Total

| Autômato | Estados (sem contar o erro) |
|---|---|
| inteiro | 2 |
| atribuição | 2 |
| ponto e vírgula | 2 |
| real | 4 |
| comentário | 3 |

---

## O nível que deu mais trabalho

Foi o **nível 4 (real)**. Na primeira tentativa eu marquei `r1` e `r3` como finais, pensando que "7" também podia ser um real. O teste do `42` quebrou, porque o `afd_real` vem antes do `afd_inteiro` nas `REGRAS`, e como os dois casavam o mesmo tamanho a ordem desempatou pro REAL. Depois de tirar o r1 dos finais, pensei em marcar o r2 também, mas aí lembrei do exemplo da oficina: se o r2 for final, `12.` é aceito. No fim ficou só o r3 como final.

Outra coisa que me travou um pouco foi juntar as duas transições do r1 (dígito e ponto). Eu tinha escrito dois `de(...)` separados no dicionário com a mesma chave `'r1'`, e o segundo sobrescrevia o primeiro. Resolvi juntando os dois dicionários com `|`.
