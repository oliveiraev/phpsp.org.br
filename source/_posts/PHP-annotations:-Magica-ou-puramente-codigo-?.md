---
createdAt: 2020-03-03
title: PHP Annotations - Mágica ou puramente código?
author: Joel Medeiros
authorEmail: jooelmedeiros+articles@gmail.com
---

Nós como desenvolvedores buscamos a melhor forma de escrever código, para isso,
utilizamos ferramentas, bibliotecas e pacotes para auxiliar nesse processo, mas
nem sempre buscamos entender o que acontece nas entranhas desses códigos
terceiros.

Com a facilidade de um `composer require`, `brew install` ou `npm install` para
instalar frameworks e bibliotecas que estão prontos para uso, não precisamos nos
preocupar em como uma funcionalidade está sendo abstraída ou quais
implementações ela segue, com isso, esse artigo busca desmistificar e
compreender uma funcionalidade que é totalmente abstraída por bibliotecas e
frameworks mas criticada por parte da comunidade, as _Mágicas Annotations_.

### Antes de tudo, afinal o que é uma annotation?

Uma definição que trago para computação é:

> "Alguma coisa que descreve o aspecto de um sujeito através de metadados em
> tempo de execução. Ou seja, os efeitos de uma Annotation sobre uma classe,
> método ou objeto somente são apenas no tempo em que é execudado" — Baseado em
> ["Annotations in PHP: They Exist" - Rafael Dohms](https://www.slideshare.net/rdohms/annotations-in-php-they-exist/)

Annotations também são conhecidas como `Decorators` e `Attributes` por outras
linguagens como Python, Javascript, C# e Rust, na presente data, o PHP chama
isso de `Annotations`.
Dessa forma, diferente do que muitos pensam, podemos afirmar que annotations não
são comentários e sim **metadados**. Leia até o final para entender por que
Annotations são **metadados** e não comentários como muitos pensam.

### Qual o uso para Annotations?

No início, o XML era muito utilizado pelas linguagens de programação para
mapeamento de metadados e comunicação entre máquinas na web, e em algum momento,
desenvolvedores buscaram algum jeito fácil de ler metadados no código, e não
apenas ter atributos de configurações, mas também transformar arquivos com
metadados em objetos e parâmetros. Com isso, no início do século, começaram a
aparecer em diversas linguagens com essa funcionalidade, como por exemplo:

- Java que teve sua primeira
  [Request For Comments (RFC) a JSR-175](https://jcp.org/en/jsr/detail?id=175)
  datada de 2002 com discussão
  `A Metadata Facility for the Java Programming Language` e posteriormente
  aprovada e implementada em 2004;
- Python em 2003 teve início da discussão de `Decorators for functions` com a
  [PEP 318](https://www.python.org/dev/peps/pep-0318/) e aprovada e implementada
  em 2004;
- Rust teve atributos modelados como atributos em 2012 seguindo a
  [ECMA-335](https://www.ecma-international.org/publications/standards/Ecma-335.htm);
- C# com o .NET core 1.1 adicionou a interface para atributos em 2016;

Hoje é utilizado para algumas coisas:

1.  Configuração
2.  Documentação de código
3.  Alterar o comportamento de um objeto em tempo de execução.

### Como funciona?

Para que uma linguagem de programação possa utilizar dos recursos de uma
annotation é necessário que ela tenha uma **engine** rodando "por debaixo dos
panos" interpretando linguagem natural em instruções computacionais. Por exemplo
a linguagem Java possui o JAPE (Java Annotation Patterns Engine) que é baseado
em expressões regulares, que consiste em um conjunto de fases, cada uma das
quais consiste em um conjunto de padrões e ações para "traduzir" linguagem
natural em instruções da linguagem, como por exemplo definir um método com
depreciado, sobrescrever uma classe, suprimir alertas e etc.

Já o PHP não tem uma engine nativa para leitura de annotations :(

Dentro do PHP Internals já houveram diversas discussões sobre incluir uma engine
nativa para traduzir annotations, mas a comunidade acredita não ser necessária,
houve inclusive em
[2010 uma proposta de RFC](https://wiki.php.net/rfc/annotations) declinada que
tratava exatamente deste tema.

**Então o PHP não tem annotations.**

**Fim do artigo.**

Pegadinha do malandro iê-ié!

Engana-se quem pensou que o PHP não possui uma forma manipular metadata, pelo
fato de não ter uma engine nativa, essa "mágica" é feita por meio de Reflections
e parsers para conseguir extrair os metadados.

### O que é uma Reflection?

> "É o processo na qual um programa de computador pode observar e modificar sua
> própria estrutura e comportamento em tempo de execução". —
> [Wikipedia](<https://en.wikipedia.org/wiki/Reflection_(computer_programming)>)

Muitas linguagens que tem a funcionalidade de Reflection em seu core, possuem as
três principais formas de seu uso, são elas instrospeção de tipo, invocação
dinâmica e visualização de metadata, essa última é um tópico importante para
entender como annotations funcionam dentro do PHP. Muita informação né? Vou
simplificar para ficar mais claro:

#### Introspecção de tipo

É a habilidade de um programa examinar a si mesmo, por exemplo, quando você
precisa validar o tipo de um objeto ou variável estaticamente.

```php
public function test($a): void {
    if ($a instanceof MyClass) {
        // do something
    }
}
```

```java
public void test(Object a) {
    if (a instanceof MyClass) {
        // do something
    }
}
```

```csharp
public void test(Object a) {
    if (a is MyClass)
    {
        // do something
    }
}

```

#### Invocação dinâmica

O processo de acessar dinamicamente propriedades, métodos ou a classe de um
objeto modificando o seu comportamento em tempo de execução:

```java
public class MyClass {
    private String a = "Some value";
}

Class<?> myClass = MyClass.class;
Object reflectionClass = myClass.newInstance();

Field reflectionProperty = reflectionClass.getClass().
            getDeclaredField("a");

reflectionProperty.setAccessible(true);
reflectionProperty.set(MyClass, "My modified parameter")

System.out.println((String) reflectionProperty.get(reflectionClass));
// My modified parameter
```

```php
class MyClass {
    private $a = 'Some value';

    /**
     * @return string
     */
    public function getA(): string {
        return $this->a;
    }
}

$myClass = new MyClass();
$reflectionProperty = new ReflectionProperty(MyClass::class, 'a');

/* It changes MyClass behavior in runtime */
$reflectionProperty->setAccessible(true);
$reflectionProperty->setValue(
	$myClass,
	'My modified parameter'
);

var_dump($myClass->getA());
// string(21) "My modified parameter"
```

#### Visualização de metadata

É a capacidade de ler metadados de qualquer tipo de documento, como classes,
tipos de parâmetros, atributos, métodos e seus parametros. Para PHP essa é a
coisa mais importante que você precisa saber quando falamos sobre annotations,
isso é usado para a leitura de metadados em `docblocks`:

```php
/**�
* A test class
*
* @param  foo bar
* @return baz
*/
class TestClass{}

$reflectionClass = new ReflectionClass('TestClass');
var_dump($reflectionClass->getDocComment());
```

O exemplo acima irá imprimir:

```
string(55) "/**
* A test class
*
* @param  foo bar
* @return baz
*/"
```

### Como Annotations funcionam no PHP?

Agora que você conhece os três principais tipos de Reflections (E espero que
você tenha lido todos eles rs), no PHP , **visualização de metadata** é usado
para reproduzir a funcionalidade de Annotations, através da aplicação de parsers
baseados em expressões regulares, para transformar linguagem natural em
linguagem computacional, ou seja, transforma metadados em variáveis,
propriedades e classes por exemplo. Vejamos um simples exemplo da aplicação de
expressões regulares para extrair metadata com PHP:

```php
/**�
* A test class
*
* @param  foo bar
* @return baz
*/
class TestClass{}

$reflectionClass = new ReflectionClass('TestClass');
$reflectionClassDocBlock = $reflectionClass->getDocComment();

preg_match_all(
    "#(@[a-zA-Z]+\s*[a-zA-Z0-9, ()_].*)#",
    $reflectionClassDocBlock,
    $matches,
    PREG_PATTERN_ORDER
);

var_dump($matches);
```

O exemplo acima irá imprimir:

```
array(2) {
  [0]=>
  array(2) {
    [0]=>
    string(15) "@param foo bar
"
    [1]=>
    string(12) "@return baz
"
  }
  [1]=>
  array(2) {
    [0]=>
    string(15) "@param foo bar
"
    [1]=>
    string(12) "@return baz
"
  }
}
```

> **NOTE**: Cada index do array `$match` tem uma parte de uma annotation e cada
> um é chamado de `token`, que pode representar a chamada de um método,
> configuração ou documentação. Esse processo de extrair informações de
> metadados e normalizá-los é chamado de **tokenização**.

Você pode estar pensando, "Mas que trabalhão para usar annotations!", não se
preocupe, PHP tem uma ótima comunidade que já criou diversas bibliotecas para
fazer esse trabalho para nós, eis algumas:

[Doctrine Annotations](https://github.com/doctrine/annotations)

[PHP Documentor](https://github.com/phpDocumentor/phpDocumentor)

[PHP Annotations](https://github.com/php-annotations)

### Então são apenas comentários?

Analisando o core do PHP vemos duas diferenças em comentários, quando escrevemos
comentários em uma única linha é interpretado como `T_COMMENT`, este é ignorado
pela engine de cache do PHP, o `opcache`.

`T_COMMENT:`

```php
// It is a comment
/* It is a comment */
# It is a comment
/*
 * It still is a comment
 */
```

E quando temos um comentário multinível, é interpretado como `T_DOC_COMMENT`,
este é lido e armazenado no cache do sistema, logo, pode ser lido em tempo de
execução através do método `ReflectionClass::getDocComment()`.

`T_DOC_COMMENT:`

```php
/**
 * It is a doc comment
 */
```

### Quem usa annotations?

[PHPUnit](https://github.com/sebastianbergmann/phpunit)

```php
use PHPUnit\Framework\TestCase;

/**
 * @backupGlobals disabled
 */
class MyTest extends TestCase
{
    /**
     * @backupGlobals enabled
     */
    public function testThatInteractsWithGlobalVariables()
    {
        // ...
    }
}
```

[Doctrine](https://github.com/doctrine/)

```php
/** @Entity */
class Message
{
    /** @Column(type="integer") */
    private $id;
    /** @Column(length=140) */
    private $text;
    /** @Column(type="datetime", name="posted_at") */
    private $postedAt;
}
```

[Symfony](https://github.com/symfony)

```php
// src/Controller/BlogController.php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class BlogController extends AbstractController
{
    /**
     * @Route("/blog", name="blog_list")
     */
    public function list()
    {
        // ...
    }
}
```

Bom agora que você entende o que são annotations, como elas funcionam e quem as
usa, eis que surge a pergunta:

### Por que usar annotations?

Prós

- Não afeta a semântica do software, ou seja, você pode injetar comportamentos
  em um objeto sem ter que extender, implementar ou instanciar um novo objeto.

- É performático pois usa o cache do sistema, como vimos anteriormente, tudo é
  armazenado em cache e lido em tempo de execução, logo, temos um ganho
  considerável de performance em relação a leitura de arquivos em disco por
  exemplo.

- É fácil de ler o código, pois sem querer (ou querendo) você cria uma
  documentação do seu código dentro dos docblocks, além de poder utilizar
  ferramentas que leem docblocks para gerar páginas de documentação como
  [PHPDocumentor](https://github.com/phpDocumentor/phpDocumentor).

- Ajuda em processos de refatoração de código segregando informações estáticas e
  ajuda a atingir algumas práticas de clean code.

Contras

- Por não afetar a semântica e poder injetar comportamentos em um objeto é mais
  difícil de debug e testes, devido ao fato de que se testa o objeto que usa as
  annotations e não é possível testar as annotations, portanto, tenha atenção de
  quais comportamentos você está inserindo em seus objetos para que no futuro
  isso não cause problemas de manutenabilidade.

- A alteração de "comentários" não deveria alterar o funcionamento de um
  software, com um olhar desapercebido ou inexperiente é possível confundir
  comentários com annotations e causar problemas ao software.
  > Pensando nisso,
  > [na mais recente RFC relacionada a Annotations (2020)](https://wiki.php.net/rfc/attributes_v2),
  > a mesma funcionalidade é chamada de **attribute** por Benjamin Eberlei, onde
  > propõe a utilização da formatação `<<...>>` ao invés da tradicional
  > `/**...*/`, que segundo ele, reduz a confusão de comentários em código para
  > iniciantes.

### Como usar annotations?

O PHP oferece apenas uma forma de consumo de metadata, o `doc-comments` como
explicado anteriormente, mas há uma vasta quantidade de bibliotecas que são
utilizadas para fazer o parser desses metadados e serem utilizados como por
exemplo para representar mapeamento de objetos relacionados (ORM).

Trago um exemplo simples utilizando a biblioteca
[doctrine annotations](https://github.com/doctrine/annotations), para entender
como aplicar ao nosso dia-a-dia. O objetivo desse exemplo é, extrair informações
de annotations de uma classe.

### Criando sua própria Annotation

Primeiro passo precisamos instalar a biblioteca
[doctrine annotations](https://github.com/doctrine/annotations) em nosso
projeto, sugiro a utilização de
[composer](https://getcomposer.org/doc/00-intro.md) para isso:

```
composer require doctrine/annotations
```

Após instalada, criaremos uma classe o qual será a annotation:

```php
namespace App\Annotation;


/**
 * Class Template
 * @package App\Annotation
 * @Annotation
 */
class Template
{
    public $label;
    public $tag;

    public function __construct(array $values)
    {
        $this->label = $values['label'];
        $this->tag = $values['tag'];
    }

    public function getLabel()
    {
        return $this->label;
    }

    public function getTag()
    {
        return $this->tag;
    }
}
```

Na classe Template é definido que é uma annotation através do docblock
`@Annotation`, esse doc-block é o que diz para a biblioteca qual classe deve ser
usada como annotation, em seguida definimos que essa annotation recebe dois
parametros `label` e `tag`.

Após definir a annotation é necessário criar outra classe que utilize essa
annotation:

```php
namespace App\Template;

use App\Annotation\Template as TemplateAnnotation;


class AnimalTextTemplate
{
    /**
     * @TemplateAnnotation(label="Characteristic 1", tag="characteristic1")
     * @return string
     */
    public function getCharacteristic1()
    {
        return 'brown';
    }

    /**
     * @TemplateAnnotation(label="Characteristic 2", tag="characteristic2")
     * @return string
     */
    public function getCharacteristic2()
    {
        return 'lazy';
    }

    /**
     * @TemplateAnnotation(label="Animal 1", tag="animal1")
     * @return string
     */
    public function getAnimal1()
    {
        return 'fox';
    }

    /**
     * @TemplateAnnotation(label="Animal 2", tag="animal2")
     * @return string
     */
    public function getAnimal2()
    {
        return 'dog';
    }
}

```

Note que os valores declarados dentro dos parentes
`(label="Characteristic 1", tag="characteristic1")` são os mesmos parametros
recebidos no construtor da annotation.

```php
// App/Annotation/Template.php

public function __construct(array $values)
{
    $this->label = $values['label'];
    $this->tag = $values['tag'];
}
```

O último passo é extrair os metadados dos atributos da classe (propriedades e
métodos), para isso é utilizada a
[ReflectionClass](https://www.php.net/manual/pt_BR/class.reflectionclass.php)
que tem função de reportar as informações de uma classe, ou seja, trazer um
relatório da classe em objeto, deixando acessível todos os metadados da classe.

```php
$reflectionClass = new \ReflectionClass(AnimalTextTemplate::class);
```

Para buscar os metadados de um método de uma classe, é utilizado o método
`getMethods()`.

```php
$reflectionClass->getMethods();
```

Por fim, cada método pode possuir mais de uma annotation, a classe
`AnnotationReader` é utilziada para fazer a distinção de qual
metadado(annotation) deve ser buscado.

```php

use Doctrine\Common\Annotations\AnnotationReader;

$annotationReader = new AnnotationReader();
$annotationReader->getMethodAnnotation($reflectionMethod, Template::class);
```

Juntando todas as partes temos o seguinte código:

```php

use Doctrine\Common\Annotations\AnnotationReader;

$reflectionClass = new \ReflectionClass(AnimalTextTemplate::class);
$annotationReader = new AnnotationReader();

$metatags = [];

foreach ($reflectionClass->getMethods(\ReflectionMethod::IS_PUBLIC) as $reflectionMethod) {
    if ($methodAnnotation = $annotationReader->getMethodAnnotation($reflectionMethod, Template::class)) {
        $metatags[] = [
            'tag' => "<%{$methodAnnotation->getTag()}%>",
            'label' => $methodAnnotation->getLabel()
        ];
    }
}

var_dump($metatags);
```

O resultado é:

```
array(4) {
  [0]=>
  array(2) {
    ["tag"]=>
    string(19) "<%characteristic1%>"
    ["label"]=>
    string(16) "Characteristic 1"
  }
  [1]=>
  array(2) {
    ["tag"]=>
    string(19) "<%characteristic2%>"
    ["label"]=>
    string(16) "Characteristic 2"
  }
  [2]=>
  array(2) {
    ["tag"]=>
    string(11) "<%animal1%>"
    ["label"]=>
    string(8) "Animal 1"
  }
  [3]=>
  array(2) {
    ["tag"]=>
    string(11) "<%animal2%>"
    ["label"]=>
    string(8) "Animal 2"
  }
}
```

#### Agora você deve estar pensando, para que raios eu irei utilizar esse pedaço de código?

Annotations como dito anteriormente, tem um vasto mundo de uso, em minha
experiência, utilizei esse trecho de código para gerar campos dinâmicos em uma
interface do usuário, e estes campos eram processados dinamicamente de acordo
com a implementação do método o qual demarca a annotation, onde as
annotations(`TemplateAnnotation`) eram os parametros dinâmicos e os valores o
retorno dos métodos. Esse exemplo de uso será explicado a seguir.

#### Possibilidade de uso

Dada uma interface com um campo de texto livre, foi disponibilizado para o
usuário diversos campos dinâmicos, esse é o retorno do método anterior, o qual
representa em `label` o campo que é exibido para o usuário, e em `tag` o valor
que será incluído no texto realmente.

```
array(4) {
  [0]=>
  array(2) {
    ["tag"]=>
    string(19) "<%characteristic1%>"
    ["label"]=>
    string(16) "Characteristic 1"
  }
  [1]=>
  array(2) {
    ["tag"]=>
    string(19) "<%characteristic2%>"
    ["label"]=>
    string(16) "Characteristic 2"
  }
  [2]=>
  array(2) {
    ["tag"]=>
    string(11) "<%animal1%>"
    ["label"]=>
    string(8) "Animal 1"
  }
  [3]=>
  array(2) {
    ["tag"]=>
    string(11) "<%animal2%>"
    ["label"]=>
    string(8) "Animal 2"
  }
}
```

Com isso o usuário pode digitar livremente o texto que quiser e incluir
metadados no texto para que seja gerada uma nova string dinamicamente.

> A frase que será utilizada será a famosa frase escondida no Microsoft Word
> 2007: `The quick brown fox jumps over the lazy dog`

O campo preenchido pelo usuário com os metadados foi o seguinte:

`The quick <%characteristic1%> <%animal1%> jumps over the <%characteristic2%> <%animal2%>`

Como definido anteriormente, na classe `AnimalTextTemplate`, é necessário
implementar que sempre ao encontrar as tags `animal1`, `animal2`,
`characteristic1` e `characteristic2` seja retornado respectivamente `fox`,
`dog`, `brown` e `lazy`.

Através de um parser de código, são definidas as regras de como ler as
annotations de uma classe. A regra utilizada será a expressão regular
`"@\<\%(.*?)\%\>@"`, ou seja, será capturado todo o conteúdo que estiver dentro
de `<%%>`.

```php
/**
 * @param string $input
 * @return array
 */
protected function getMetatags(string $input): array
{
    if (preg_match_all("@\<\%(.*?)\%\>@", $input, $match)) {
        return $match;
    }

    return [];
}
```

O resultado do método anterior, recebendo o input do usuário será o seguinte:

```
array(2) {
  [0]=>
  array(4) {
    [0]=>
    string(19) "<%characteristic1%>"
    [1]=>
    string(11) "<%animal1%>"
    [2]=>
    string(19) "<%characteristic2%>"
    [3]=>
    string(11) "<%animal2%>"
  }
  [1]=>
  array(4) {
    [0]=>
    string(15) "characteristic1"
    [1]=>
    string(7) "animal1"
    [2]=>
    string(15) "characteristic2"
    [3]=>
    string(7) "animal2"
  }
}
```

No primeiro índice temos todas as metatags capturadas, no segundo, as tags
nominadas. Com isso, é possível acessar a classe o qual definiu essas tags para
extrair os metadados:

```php
use Doctrine\Common\Collections\ArrayCollection;

/**
 * @param array $metadataKeys
 * @return ArrayCollection
 * @throws \Exception
 */
protected function getMetadata(array $metadataKeys): ArrayCollection
{
    $metadata = new ArrayCollection();

    foreach ($metadataKeys as $metadataKey) {
        $method = "get" . lcfirst(str_replace('_', '', ucwords(strtolower($metadataKey), '_')));

        if (!method_exists($this, $method)) {
            throw new \Exception("Method {$method} not implemented in " . get_class($this) . " class");
        }

        $metadata->set($metadataKey, $this->$method());
    }

    return $metadata;
}
```

O output desse trecho de código irá nos trazer um conjunto de dados associados
pela tag e seu real valor:

```
object(Doctrine\Common\Collections\ArrayCollection)#2 (1) {
  ["elements":"Doctrine\Common\Collections\ArrayCollection":private]=>
  array(4) {
    ["characteristic1"]=>
    string(5) "brown"
    ["animal1"]=>
    string(3) "fox"
    ["characteristic2"]=>
    string(4) "lazy"
    ["animal2"]=>
    string(3) "dog"
  }
}
```

Com esses dados, podemos fazer uma simples regra de substituição de parametros e
aplicar na string do usuário:

```php
use Doctrine\Common\Collections\ArrayCollection;

$metatags = $this->getMetatags($input);
$findBy = $metatags[0];
$metadataKeys = $metatags[1];

/** @var ArrayCollection $metadata */
$metadata = $this->getMetadata($metadataKeys);

$output = $input;

foreach ($findBy as $key => $value) {
    $metadataKey = $metadataKeys[$key];
    if (!$metadata->containsKey($metadataKey)) {
        continue;
    }

    $output = str_replace($value, $metadata->get($metadataKey), $output);
}

var_dump($output);
```

Output:

```
string(50) "The quick brown fox jumps over the lazy dog typing"
```

### Conclusão

O _PHP não possui_ uma engine nativa de Annotations, mas a necessidade de uma é
dispensável visto que temos acesso a todos os metadados de uma classe através da
`ReflectionClass`, a função dos parsers nesse contexto é vital para que esses
dados possam ser utilizados em nossos códigos. A vasta quantidade de bibliotecas
para Docblock parsers disponíveis leva a compreender a decisão do PHP Internals
de recusar a implantação de uma engine nativa para annotations.

O uso de annotations é amplamente discutido em diversas comunidades e difundido
por diversas ferramentas, mas seu uso não se limita apenas a essas "magias" das
ferramentas, entendendo o seu uso e aplicabilidade serve também para resolver
problemas do dia-a-dia.

O código desse artigo encontra-se disponível em:

https://github.com/joelmedeiros/useful-annotations

### Referências

[Attribute Interface - C#](https://docs.microsoft.com/pt-br/dotnet/api/system.runtime.interopservices._attribute?view=netframework-4.8)

[Attributes - Rust](https://doc.rust-lang.org/reference/attributes.html)

[Annotations in PHP: They Exist - Rafael Dohms](https://www.slideshare.net/rdohms/annotations-in-php-they-exist/138-Reflection)

[ECMA-335 - Common Language Infrastructure (CLI)](https://www.ecma-international.org/publications/standards/Ecma-335.htm)

[How Do Annotations Work in Java? - Yashwant Golecha](https://dzone.com/articles/how-annotations-work-java)

[Java Annotations](https://www.javatpoint.com/java-annotation)

[Java Reflection - Private Fields and Methods](http://tutorials.jenkov.com/java-reflection/private-fields-and-methods.html)

[JSR-175 - JSR 175: A Metadata Facility for the Java Programming Language](https://jcp.org/en/jsr/detail?id=175)

[PHP RFC: Annotations 2.0](https://wiki.php.net/rfc/annotations_v2)

[PHP RFC: Attributes 2.0](https://wiki.php.net/rfc/attributes_v2)

[PHP: Annotations are an Abomination](https://r.je/php-annotations-are-an-abomination)

[PEP 318 - Decorators for Functions and Methods](https://www.python.org/dev/peps/pep-0318/)

[Reflections - Rafael Uchôa](https://pt.slideshare.net/rafaeluchoa/reflections-15781359)

[Should we use PHP Annotations?](https://medium.com/a-young-devoloper/should-we-use-php-annotations-4efafea23334)

[Understanding annotations](https://php-annotations.readthedocs.io/en/latest/UsingAnnotations.html)

[USING ANNOTATIONS IN PHP WITH DOCTRINE ANNOTATION READER](http://masnun.com/2012/08/12/using-annotations-in-php-with-doctrine-annotation-reader.html)
