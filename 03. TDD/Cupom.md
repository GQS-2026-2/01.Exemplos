# 🎟️ Exemplo 04 — Validação de Cupom Promocional com TDD

## 🎯 Objetivo

Desenvolver uma classe Java chamada `CupomService` para verificar se um cupom promocional é válido. A evolução deve acontecer **um teste de cada vez**, seguindo o ciclo **🔴 RED → 🟢 GREEN → 🔵 REFACTOR**.

## 📋 Regras de negócio

- O cupom `DESC10` é válido.
- O cupom `FRETEGRATIS` também é válido.
- Qualquer outro código é inválido.
- `null` e texto vazio (inclusive contendo apenas espaços) são inválidos e devem lançar `IllegalArgumentException`.
- A comparação diferencia letras maiúsculas e minúsculas: `desc10` não é igual a `DESC10`.

## 🗂️ Estrutura do projeto Maven

```text
exemplo-04-cupom/
├── pom.xml
└── src/
    ├── main/java/br/edu/tdd/CupomService.java
    └── test/java/br/edu/tdd/CupomServiceTest.java
```

Para usar JUnit 5, configure o `pom.xml` conforme o arquivo-modelo ao final deste material. Os trechos abaixo substituem o conteúdo da classe ou acrescentam métodos de teste **progressivamente**; não são classes independentes para executar ao mesmo tempo.

---

# 🔁 Ciclo 1 — Validar o cupom DESC10

## 🔴 RED — Escrever o primeiro teste

Crie `CupomServiceTest.java`:

```java
package br.edu.tdd;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CupomServiceTest {

    @Test
    void deveAceitarCupomDesc10() {
        CupomService service = new CupomService();
        assertTrue(service.ehValido("DESC10"));
    }
}
```

Ao executar `mvn test`, a classe `CupomService` ainda não existe: **o projeto não compila**. Isso é esperado durante a criação da interface, mas precisamos também observar uma falha de comportamento.

Crie `CupomService.java` com uma implementação provisória:

```java
package br.edu.tdd;

public class CupomService {
    public boolean ehValido(String codigo) {
        return false;
    }
}
```

Execute novamente:

```bash
mvn test
```

**Resultado esperado:** 🔴 falha de asserção, pois o teste espera `true`, mas recebe `false`.

## 🟢 GREEN — Fazer o teste passar

Altere **somente** o método:

```java
public boolean ehValido(String codigo) {
    return true;
}
```

Execute `mvn test`: 🟢 **1 teste passando**. O retorno fixo é propositalmente simples, mas ainda não atende às demais regras.

## 🔵 REFACTOR — Revisar o código

Ainda não existe duplicação ou estrutura complexa a melhorar. **Não é obrigatório mudar o código em toda refatoração.** Mantenha o teste verde e avance.

---

# 🔁 Ciclo 2 — Rejeitar cupom desconhecido

## 🔴 RED — Adicionar o segundo teste

Dentro de `CupomServiceTest`, adicione:

```java
@Test
void deveRejeitarCupomDesconhecido() {
    CupomService service = new CupomService();
    assertFalse(service.ehValido("PROMO50"));
}
```

Execute `mvn test`: 🔴 o novo teste falha porque a implementação retorna `true` para qualquer cupom.

## 🟢 GREEN — Evoluir a implementação

Substitua o método por:

```java
public boolean ehValido(String codigo) {
    return "DESC10".equals(codigo);
}
```

Execute `mvn test`: 🟢 **2 testes passando**. Usar a constante literal à esquerda do `equals` evita `NullPointerException` se o argumento for `null`; a política de erro para `null` será acrescentada em outro ciclo.

## 🔵 REFACTOR — Revisar

O método permanece curto e legível. Preserve-o sem mudanças.

---

# 🔁 Ciclo 3 — Aceitar FRETEGRATIS

## 🔴 RED — Acrescentar um novo comportamento

```java
@Test
void deveAceitarCupomFreteGratis() {
    CupomService service = new CupomService();
    assertTrue(service.ehValido("FRETEGRATIS"));
}
```

Execute `mvn test`: 🔴 falha para `FRETEGRATIS`.

## 🟢 GREEN — Atender aos dois cupons

```java
public boolean ehValido(String codigo) {
    return "DESC10".equals(codigo)
        || "FRETEGRATIS".equals(codigo);
}
```

Execute `mvn test`: 🟢 **3 testes passando**.

## 🔵 REFACTOR — Dar nome aos valores de negócio

Agora os códigos válidos estão embutidos no método. Extraia constantes:

```java
private static final String CUPOM_DESCONTO = "DESC10";
private static final String CUPOM_FRETE = "FRETEGRATIS";

public boolean ehValido(String codigo) {
    return CUPOM_DESCONTO.equals(codigo)
        || CUPOM_FRETE.equals(codigo);
}
```

Execute `mvn test`: 🟢 **3 testes continuam passando**.

---

# 🔁 Ciclo 4 — Tratar código vazio e null

## 🔴 RED — Validar entradas inválidas

Adicione dois testes, executando cada um antes de incluir o próximo:

```java
@Test
void deveLancarExcecaoParaCupomVazio() {
    CupomService service = new CupomService();
    assertThrows(IllegalArgumentException.class,
        () -> service.ehValido("   "));
}
```

🔴 O método retorna `false`, mas o teste espera uma exceção.

## 🟢 GREEN — Corrigir o caso vazio

Insira no começo de `ehValido`:

```java
if (codigo != null && codigo.isBlank()) {
    throw new IllegalArgumentException("Cupom não pode ser vazio");
}
```

Execute `mvn test`: 🟢 **4 testes passando**.

## 🔴 RED — Agora o caso null

```java
@Test
void deveLancarExcecaoParaCupomNulo() {
    CupomService service = new CupomService();
    assertThrows(IllegalArgumentException.class,
        () -> service.ehValido(null));
}
```

🔴 O método retorna `false`, sem lançar exceção.

## 🟢 GREEN — Consolidar a validação

Substitua a condição por:

```java
if (codigo == null || codigo.isBlank()) {
    throw new IllegalArgumentException("Cupom deve ser informado");
}
```

Execute `mvn test`: 🟢 **5 testes passando**.

## 🔵 REFACTOR — Versão final da classe

```java
package br.edu.tdd;

public class CupomService {

    private static final String CUPOM_DESCONTO = "DESC10";
    private static final String CUPOM_FRETE = "FRETEGRATIS";

    public boolean ehValido(String codigo) {
        if (codigo == null || codigo.isBlank()) {
            throw new IllegalArgumentException("Cupom deve ser informado");
        }

        return CUPOM_DESCONTO.equals(codigo)
            || CUPOM_FRETE.equals(codigo);
    }
}
```

Execute `mvn test`: 🟢 **5 testes passando**. A versão final da classe é apresentada integralmente; mantenha na classe de testes os cinco métodos adicionados nos ciclos.

---

## ⚙️ `pom.xml` mínimo (Java 21 + JUnit Jupiter)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>br.edu.tdd</groupId>
    <artifactId>exemplo-04-cupom</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.release>21</maven.compiler.release>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.11.4</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.13.0</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.2</version>
            </plugin>
        </plugins>
    </build>
</project>
```
