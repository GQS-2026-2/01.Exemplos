# 🧪 Projeto Calculadora com Cucumber em Java

Este projeto demonstra como utilizar **Java**, **JUnit 5** e **Cucumber** para automatizar **testes de aceitação (BDD)**, aplicando cenários baseados em comportamento para operações matemáticas de uma calculadora.

---

## ✅ Pré-requisitos

Certifique-se de ter instalado:

| Ferramenta | Versão recomendada |
|-----------|-------------------|
| Java | **21** |
| Visual Studio Code (VS Code) | Versão estável |
| Maven | **3.9+** |
| Cucumber for Java | (Incluído via dependências Maven) |

No VS Code, instale as extensões **Extension Pack for Java** e **Maven for Java**. A extensão **Cucumber (Gherkin) Full Support** é opcional e auxilia na edição dos arquivos `.feature`.

Verifique a versão do Java:
```bash
java -version
```

## 📁 Estrutura do Projeto
```text
projeto-cucumber
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── calculadora
    │           └── Calculadora.java
    └── test
        ├── java
        │   └── steps
        │       ├── CalculadoraSteps.java
        │       └── RunnerTest.java
        └── resources
            └── features
                └── calculadora.feature
```                


## 📝 Feature (calculadora.feature)

```gherkin
# language: pt

Funcionalidade: Calcular operações matemáticas

  Esquema do Cenário: mostrar o resultado quando executar uma operação na calculadora
    Dado que a calculadora está disponível
    Quando calcular <numero1> e <numero2> usando "<operacao>"
    Então o resultado deve ser <resultado>

    Exemplos:
      | numero1 | numero2 | operacao      | resultado |
      | 4       | 2       | soma          | 6         |
      | 4       | 2       | subtração     | 2         |
      | 4       | 2       | multiplicação | 8         |
      | 4       | 2       | divisão       | 2         |
      | 0       | 5       | soma          | 5         |
      | 5       | 0       | subtração     | 5         |
```
## 🔧 Implementação da Classe Calculadora.java
```java
package calculadora;

public class Calculadora {
    public int calcular(int numero1, int numero2, String operacao) {
        return switch (operacao) {
            case "soma" -> numero1 + numero2;
            case "subtração" -> numero1 - numero2;
            case "multiplicação" -> numero1 * numero2;
            case "divisão" -> {
                if (numero2 == 0) {
                    throw new ArithmeticException("Divisão por zero não é permitida");
                }
                yield numero1 / numero2;
            }
            default -> throw new IllegalArgumentException("Operação inválida: " + operacao);
        };
    }
}
```

## 🧩 Steps Definitions (CalculadoraSteps.java)

```java
package steps;

import calculadora.Calculadora;
import io.cucumber.java.pt.*;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class CalculadoraSteps {

    private int resultado;
    private Calculadora calculadora;

    @Dado("que a calculadora está disponível")
    public void calculadoraDisponivel() {
        calculadora = new Calculadora();
    }

    @Quando("calcular {int} e {int} usando {string}")
    public void calcular(int numero1, int numero2, String operacao) {
        resultado = calculadora.calcular(numero1, numero2, operacao);
    }

    @Entao("o resultado deve ser {int}")
    public void verificarResultado(int esperado) {
        assertEquals(esperado, resultado);
    }
}
```

## 🧭 Runner (JUnit 5) — RunnerTest.java

```java
package steps;

import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

import static io.cucumber.junit.platform.engine.Constants.PLUGIN_PROPERTY_NAME;
import static io.cucumber.junit.platform.engine.Constants.GLUE_PROPERTY_NAME;

@Suite
@IncludeEngines("cucumber")
@SelectClasspathResource("features")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "steps")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, value = "pretty")
public class RunnerTest {
}
```
## ⚙️ Dependências (pom.xml atualizado)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>exemplo</groupId>
<artifactId>calculadora-bdd</artifactId>
<version>1.0-SNAPSHOT</version>
<properties>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
<dependencies>
    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-java</artifactId>
        <version>7.20.1</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>io.cucumber</groupId>
        <artifactId>cucumber-junit-platform-engine</artifactId>
        <version>7.20.1</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.11.3</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-suite</artifactId>
        <version>1.11.3</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
</project>
```
## 💻 Abrir o projeto no VS Code

1. Crie uma pasta chamada `projeto-cucumber` e monte a estrutura mostrada acima.
2. Abra o VS Code e selecione **Arquivo → Abrir Pasta** (ou **File → Open Folder**).
3. Escolha a pasta que contém o arquivo `pom.xml`.
4. Aguarde a importação do projeto Maven pelo suporte a Java.
5. Confirme no terminal integrado que `java -version` e `mvn -version` reconhecem as ferramentas.

## ▶️ Como executar os testes

Abra **Terminal → Novo Terminal** no VS Code. Dentro da pasta do projeto, execute:
```bash
mvn test
```
Os resultados aparecem no terminal. Você também pode utilizar o painel **Testing** do VS Code quando os testes forem descobertos pela extensão Java instalada.

{% hint style="info" %}
💡 Este exemplo utiliza **divisão inteira**: `5 / 2` resulta em `2`. Para resultados decimais, altere os tipos e os cenários para `double` ou `BigDecimal`. A divisão por zero lança `ArithmeticException` e pode ser explorada em um cenário adicional.
{% endhint %}
