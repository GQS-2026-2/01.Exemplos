# 📦 Exemplo 05 — Controle de Estoque com TDD

## 🎯 Objetivo

Desenvolver uma classe `Estoque` que controle a quantidade de unidades disponíveis de um produto. Vamos praticar TDD em um objeto que **altera seu estado** ao longo do tempo.

## 📋 Regras de negócio

- O estoque começa com **zero** unidades.
- É possível adicionar unidades com `adicionar(int quantidade)`.
- É possível retirar unidades com `retirar(int quantidade)`.
- Não é permitido adicionar ou retirar **zero** ou valores negativos.
- Não é permitido retirar mais unidades do que as disponíveis.
- Uma operação inválida **não pode alterar o saldo**.

## 🗂️ Estrutura do projeto Maven

```text
exemplo-05-estoque/
├── pom.xml
└── src/
    ├── main/java/br/edu/tdd/Estoque.java
    └── test/java/br/edu/tdd/EstoqueTest.java
```

Use o `pom.xml` do exemplo 04, trocando o `artifactId` para `exemplo-05-estoque`. Todos os trechos de testes são acumulados na **mesma classe `EstoqueTest`**. As implementações dos métodos mudam em cada ciclo; substitua versões antigas, não as duplique.

---

# 🔁 Ciclo 1 — Estoque inicial zero

## 🔴 RED — Escrever o teste

Crie `EstoqueTest.java`:

```java
package br.edu.tdd;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class EstoqueTest {

    @Test
    void deveIniciarComQuantidadeZero() {
        Estoque estoque = new Estoque();
        assertEquals(0, estoque.getQuantidade());
    }
}
```

Execute `mvn test`: 🔴 classe `Estoque` inexistente, portanto o código ainda não compila.

Crie a classe com um retorno provisório incorreto, para observar a falha de comportamento:

```java
package br.edu.tdd;

public class Estoque {
    public int getQuantidade() {
        return -1;
    }
}
```

🔴 **Esperado `0`, obtido `-1`**.

## 🟢 GREEN — Implementar o mínimo

```java
public int getQuantidade() {
    return 0;
}
```

Execute `mvn test`: 🟢 **1 teste passando**.

## 🔵 REFACTOR — Revisar

Não há nada importante a simplificar. Mantenha a solução e avance.

---

# 🔁 Ciclo 2 — Adicionar unidades

## 🔴 RED — Escrever o teste

```java
@Test
void deveAdicionarCincoUnidades() {
    Estoque estoque = new Estoque();
    estoque.adicionar(5);
    assertEquals(5, estoque.getQuantidade());
}
```

🔴 O método `adicionar` ainda não existe. Crie-o provisoriamente:

```java
public void adicionar(int quantidade) {
    // Ainda não implementado
}
```

Execute `mvn test`: 🔴 esperado `5`, obtido `0`.

## 🟢 GREEN — Passar o teste

Acrescente um atributo e altere os métodos:

```java
private int quantidade = 0;

public int getQuantidade() {
    return quantidade;
}

public void adicionar(int valor) {
    quantidade += valor;
}
```

Execute `mvn test`: 🟢 **2 testes passando**.

## 🔵 REFACTOR — Revisar

O atributo privado representa o estado; o método `getQuantidade` permite consultá-lo. Não é necessário refatorar ainda.

---

# 🔁 Ciclo 3 — Retirar unidades

## 🔴 RED — Escrever o teste

```java
@Test
void deveRetirarDuasUnidades() {
    Estoque estoque = new Estoque();
    estoque.adicionar(5);
    estoque.retirar(2);
    assertEquals(3, estoque.getQuantidade());
}
```

🔴 O método `retirar` não existe. Crie uma versão provisória:

```java
public void retirar(int valor) {
    // Ainda não implementado
}
```

Execute `mvn test`: 🔴 esperado `3`, obtido `5`.

## 🟢 GREEN — Implementar a retirada

```java
public void retirar(int valor) {
    quantidade -= valor;
}
```

Execute `mvn test`: 🟢 **3 testes passando**.

## 🔵 REFACTOR — Revisar

O código está simples, mas ainda permite operações inválidas. Essas novas regras serão desenvolvidas nos ciclos seguintes, guiadas por testes.

---

# 🔁 Ciclo 4 — Impedir retirada maior que o saldo

## 🔴 RED — Criar o teste

```java
@Test
void deveImpedirRetiradaMaiorQueEstoque() {
    Estoque estoque = new Estoque();
    estoque.adicionar(3);

    assertThrows(IllegalArgumentException.class,
        () -> estoque.retirar(4));
    assertEquals(3, estoque.getQuantidade());
}
```

🔴 A implementação anterior subtrai `4` de `3`, deixando saldo `-1`, e não lança exceção.

## 🟢 GREEN — Validar a quantidade disponível

```java
public void retirar(int valor) {
    if (valor > quantidade) {
        throw new IllegalArgumentException("Estoque insuficiente");
    }
    quantidade -= valor;
}
```

Execute `mvn test`: 🟢 **4 testes passando**.

## 🔵 REFACTOR — Revisar

A verificação ocorre **antes** da alteração de estado. Assim, uma tentativa inválida não modifica o saldo.

---

# 🔁 Ciclo 5 — Impedir adição de zero ou negativo

## 🔴 RED — Criar um teste para zero

```java
@Test
void deveRejeitarAdicaoDeZero() {
    Estoque estoque = new Estoque();
    assertThrows(IllegalArgumentException.class,
        () -> estoque.adicionar(0));
    assertEquals(0, estoque.getQuantidade());
}
```

🔴 Nenhuma exceção é lançada.

## 🟢 GREEN — Validar adição

```java
public void adicionar(int valor) {
    if (valor <= 0) {
        throw new IllegalArgumentException("Quantidade deve ser positiva");
    }
    quantidade += valor;
}
```

Execute `mvn test`: 🟢 **5 testes passando**.

## 🔴 RED? — Acrescentar teste para negativo

```java
@Test
void deveRejeitarAdicaoNegativa() {
    Estoque estoque = new Estoque();
    assertThrows(IllegalArgumentException.class,
        () -> estoque.adicionar(-2));
    assertEquals(0, estoque.getQuantidade());
}
```

🟢 **Esse teste já passa!** A condição `valor <= 0` também cobre negativos. Isso mostra que nem todo teste adicional provoca uma nova falha. Para um ciclo TDD estrito, poderíamos ter escolhido primeiro um caso que a implementação ainda não atendesse.

---

# 🔁 Ciclo 6 — Impedir retirada de zero ou negativo

## 🔴 RED — Criar o teste

```java
@Test
void deveRejeitarRetiradaDeZero() {
    Estoque estoque = new Estoque();
    estoque.adicionar(5);

    assertThrows(IllegalArgumentException.class,
        () -> estoque.retirar(0));
    assertEquals(5, estoque.getQuantidade());
}
```

🔴 O método atual não lança exceção.

## 🟢 GREEN — Adicionar validação

```java
public void retirar(int valor) {
    if (valor <= 0) {
        throw new IllegalArgumentException("Quantidade deve ser positiva");
    }
    if (valor > quantidade) {
        throw new IllegalArgumentException("Estoque insuficiente");
    }
    quantidade -= valor;
}
```

Execute `mvn test`: 🟢 **7 testes passando**.

Adicione também:

```java
@Test
void deveRejeitarRetiradaNegativa() {
    Estoque estoque = new Estoque();
    estoque.adicionar(5);

    assertThrows(IllegalArgumentException.class,
        () -> estoque.retirar(-1));
    assertEquals(5, estoque.getQuantidade());
}
```

🟢 **8 testes passando**. A implementação já atende a esse caso.

---

# 🔵 REFACTOR FINAL — Eliminar validação duplicada

Os métodos `adicionar` e `retirar` repetem a verificação de quantidade positiva. Podemos extrair essa verificação para um método privado, **sem alterar o comportamento**.

**Versão final completa de `Estoque.java`:**

```java
package br.edu.tdd;

public class Estoque {

    private int quantidade;

    public int getQuantidade() {
        return quantidade;
    }

    public void adicionar(int valor) {
        validarQuantidadePositiva(valor);
        quantidade += valor;
    }

    public void retirar(int valor) {
        validarQuantidadePositiva(valor);

        if (valor > quantidade) {
            throw new IllegalArgumentException("Estoque insuficiente");
        }

        quantidade -= valor;
    }

    private void validarQuantidadePositiva(int valor) {
        if (valor <= 0) {
            throw new IllegalArgumentException("Quantidade deve ser positiva");
        }
    }
}
```

Execute:

```bash
mvn test
```

**Resultado esperado:** 🟢 **8 testes passando** e nenhuma alteração nas regras de negócio. O arquivo `EstoqueTest.java` deve conter os oito métodos adicionados durante os ciclos.
