🚚 Exemplo — calcular frete com TDD
Regras: pedidos abaixo de R$ 100 pagam R$ 15 de frete; pedidos a partir de R$ 100 têm frete grátis; valores negativos são inválidos.

🛠️ Passo 0 — preparar o projeto
Pré-requisitos: JDK 21, Maven 3.9+ e VS Code (com Extension Pack for Java) ou NetBeans.

Crie um projeto Maven denominado tdd-frete com a estrutura:

```text
tdd-exemplos/
├── pom.xml
└── src/
    ├── main/java/br/edu/gqs/
        ├── FreteService.java
    └── test/java/br/edu/gqs/
        ├── FreteServiceTest.java
```

Use este pom.xml de exemplo:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>br.edu.gqs</groupId>
    <artifactId>tdd-exemplos</artifactId>
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
    <build><plugins>
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
    </plugins></build>
</project>
```

Confira no terminal:

```bash
java -version
mvn -version
mvn test
```

Uma execução inicial sem testes pode terminar com sucesso. Isso não comprova que há testes suficientes.

🔴 Ciclo 1 — RED: pedido com frete grátis

Crie FreteServiceTest.java:

```java
package br.edu.gqs;

import static org.junit.jupiter.api.Assertions.*;
import org.junit.jupiter.api.Test;

class FreteServiceTest {
    @Test
    void deveDarFreteGratisParaPedidoAcimaDoLimite() {
        assertEquals(0.0, new FreteService().calcular(150.0));
    }
}
```

Crie FreteService.java com assinatura inicial (provisória):

```java
package br.edu.gqs;

public class FreteService {
    public double calcular(double valorPedido) {
        return 15.0;
    }
}
```

Execute mvn test: esperado 0.0, obtido 15.0 → RED.
🟢 Ciclo 1 — GREEN
Troque o retorno provisoriamente para `return 0.0;` e execute `mvn test`. O primeiro teste passa.
🔴 Ciclo 2 — RED: pedido abaixo do limite
Adicione à classe de testes:

```java
@Test
void deveCobrarFreteParaPedidoAbaixoDoLimite() {
    assertEquals(15.0, new FreteService().calcular(80.0));
}
```

Execute `mvn test`: falha esperada, pois o método ainda retorna `0.0`.
🟢 Ciclo 2 — GREEN
Atualize o método:

```java
public double calcular(double valorPedido) {
    return valorPedido >= 100.0 ? 0.0 : 15.0;
}
```

Execute `mvn test`: os dois testes passam.
🔴 Ciclo 3 — testar valor-limite
Adicione:

```java
@Test
void deveDarFreteGratisExatamenteEmCemReais() {
    assertEquals(0.0, new FreteService().calcular(100.0));
}
```

O teste já deverá passar: esse é um teste de fronteira e regressão, mas não inicia um novo ciclo RED. Para praticar TDD estrito, acrescente um comportamento ainda não implementado.
🔴 Ciclo 4 — RED: valor negativo
Adicione:

```java
@Test
void deveRejeitarPedidoNegativo() {
    assertThrows(IllegalArgumentException.class,
            () -> new FreteService().calcular(-1.0));
}
```

Execute `mvn test`: o teste falhará porque nenhuma exceção é lançada.
🟢 Ciclo 4 — GREEN
Versão final de FreteService.java:

```java
package br.edu.gqs;

public class FreteService {
    public double calcular(double valorPedido) {
        if (valorPedido < 0) {
            throw new IllegalArgumentException("Valor do pedido não pode ser negativo");
        }
        return valorPedido >= 100.0 ? 0.0 : 15.0;
    }
}
```

Execute `mvn test`: os quatro testes devem passar.
🔵 REFACTOR
Confira se nomes deixam claras as regras, se não há duplicação e se todos os testes continuam passando. Opcional: refatore o operador ternário para um if se a turma considerar mais legível. Nunca mude o resultado da regra só para “melhorar” o código.
Observação técnica: double é suficiente para ilustrar a regra fixa. Para valores monetários reais que envolvem cálculos e arredondamento, prefira BigDecimal com política de arredondamento explícita.