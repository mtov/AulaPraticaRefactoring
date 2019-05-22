# Aula Prática sobre Refactoring

**Prof. Marco Tulio Valente**

Objetivo: realizar alguns refactorings em um sistema hipotético, usado no livro do Fowler. Em caso de dúvidas, consulte o Capítulo 1 desse livro.


Instruções:

* Primeiro, crie um repositório no GitHub.

* Vá seguindo o roteiro, refactoring a refactoring.

* Após cada refactoring, dê um **COMMIT & PUSH**. Esses commits serão usados na correção, para garantir que realizou todos os refactorings solicitados.


# Versão Inicial

As classes que vamos usar fazem parte de um sistema de video-locadora, para aluguel de vídeos.

Inicialmente, são três classes: `Movie` (filmes que podem ser alugados), `Rental` (dados de um aluguel) e `Customer` (clientes da locadora).


```java
public class Movie {

  public static final int  CHILDRENS = 2;
  public static final int  REGULAR = 0;
  public static final int  NEW_RELEASE = 1;

  private String _title;
  private int _priceCode;

  public Movie(String title, int priceCode) {
      _title = title;
      _priceCode = priceCode;
  }

  public int getPriceCode() {
      return _priceCode;
  }

  public void setPriceCode(int arg) {
     _priceCode = arg;
  }

  public String getTitle (){
      return _title;
  };
}

class Rental {
    private Movie _movie;
    private int _daysRented;

    public Rental(Movie movie, int daysRented) {
      _movie = movie;
      _daysRented = daysRented;
    }
    public int getDaysRented() {
      return _daysRented;
    }
    public Movie getMovie() {
      return _movie;
    }
}

class Customer {
   private String _name;
   private Vector _rentals = new Vector();

   public Customer (String name){
      _name = name;
   };

   public void addRental(Rental arg) {
      _rentals.addElement(arg);
   }
   public String getName (){
      return _name;
   };
  
  public String statement() {
     double totalAmount = 0;
     int frequentRenterPoints = 0;
     Enumeration rentals = _rentals.elements();
     String result = "Rental Record for " + getName() + "\n";
     while (rentals.hasMoreElements()) {
        double thisAmount = 0;
        Rental each = (Rental) rentals.nextElement();

        //determine amounts for each line
        switch (each.getMovie().getPriceCode()) {
           case Movie.REGULAR:
              thisAmount += 2;
              if (each.getDaysRented() > 2)
                 thisAmount += (each.getDaysRented() - 2) * 1.5;
              break;
           case Movie.NEW_RELEASE:
              thisAmount += each.getDaysRented() * 3;
              break;
           case Movie.CHILDRENS:
              thisAmount += 1.5;
              if (each.getDaysRented() > 3)
                 thisAmount += (each.getDaysRented() - 3) * 1.5;
               break;
        }

        // add frequent renter points
        frequentRenterPoints ++;
        // add bonus for a two day new release rental
        if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) &&
            each.getDaysRented() > 1) frequentRenterPoints ++;

        //show figures for this rental
        result += "\t" + each.getMovie().getTitle()+ "\t" +
            String.valueOf(thisAmount) + "\n";
        totalAmount += thisAmount;

     }
     //add footer lines
     result +=  "Amount owed is " + String.valueOf(totalAmount) + "\n";
     result += "You earned " + String.valueOf(frequentRenterPoints) +
             " frequent renter points";
     return result;
}
```
**COMMIT & PUSH**

# Teste de Unidade

Implemente um teste para o método `statement`. Crie alguns objetos do tipo `Movie`; crie um `Customer` com alguns aluguéis (isto é, objetos do tipo `Rental`) e implemente um teste de unidade. Esse teste deve checar se a string retornada por `statement` é realmente aquela esperada.

Após cada refactoring deste roteiro (e antes de dar um push/commit), se certifique de que o teste criado continua passando.

**Commit & Push**

# Refactorig 1: Extract Method

Extrair um método, chamado `amountFor` de `Customer.statement()`; já que esse último é um método maior e que faz muitas coisas. O método extraído vai conter o código relativo ao comentário *determine amounts for each line*.

Após o Extract Method, o código de `statement` será:

```java
public String statement() {
   double totalAmount = 0;
   int frequentRenterPoints = 0;
   Enumeration rentals = _rentals.elements();
   String result = "Rental Record for " + getName() + "\n";
   while (rentals.hasMoreElements()) {
      double thisAmount = 0;
      Rental each = (Rental) rentals.nextElement();

      thisAmount = amountFor(each);

      // add frequent renter points
      frequentRenterPoints ++;
      // add bonus for a two day new release rental
      if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) &&
         each.getDaysRented() > 1) frequentRenterPoints ++;

      //show figures for this rental
      result += "\t" + each.getMovie().getTitle()+ "\t" +
          String.valueOf(thisAmount) + "\n";
      totalAmount += thisAmount;
   } 
   //add footer lines
   result +=  "Amount owed is " + String.valueOf(totalAmount) + "\n";
   result += "You earned " + String.valueOf(frequentRenterPoints) +
            " frequent renter points";
   return result;
}
```

**Commit & Push**

# Refactoring 2: Rename

Renomear o parâmetro de `amounfFor` para ter o nome `aRental`. Veja abaixo a versão após a renomeação:

```java
private double amountFor(Rental aRental) {
   double result = 0;
   switch (aRental.getMovie().getPriceCode()) {
      case Movie.REGULAR:
         result += 2;
         if (aRental.getDaysRented() > 2)
            result += (aRental.getDaysRented() - 2) * 1.5;
         break;
      case Movie.NEW_RELEASE:
         result += aRental.getDaysRented() * 3;
         break;
      case Movie.CHILDRENS:
         result += 1.5;
         if (aRental.getDaysRented() > 3)
            result += (aRental.getDaysRented() - 3) * 1.5;
         break;
   }
   return result;
}
```

**Commit & Push**

# Refactoring 3: Move Method

Mover o método `amountFor(Rental)` da classe `Customer` para a classe `Rental`, já que esse método não usa informações da primeira, mas sim da segunda classe. 

Inicialmente, mova esse método para `Rental`, mas com o nome `getCharge()`; a versão antiga vai ser alterada para apenas delegar a chamada, para o método movido.  A ideia é que refactorings devem ser feitos em pequenos passos, para garantir que nada está sendo quebrado.

```java
class Rental...
   double getCharge() { // veja que não precisa mais de parâmetro
     double result = 0;
     switch (getMovie().getPriceCode()) {
        case Movie.REGULAR:
           result += 2;
           if (getDaysRented() > 2)
              result += (getDaysRented() - 2) * 1.5;
           break;
        case Movie.NEW_RELEASE:
           result += getDaysRented() * 3;
           break;
        case Movie.CHILDRENS:
           result += 1.5;
           if (getDaysRented() > 3)
              result += (getDaysRented() - 3) * 1.5;
           break;
     }
     return result;
}

class Customer...
   private double amountFor(Rental aRental) {
      return aRental.getCharge();  // agora apenas delega chamada para método movido
   }

```

Estando tudo funcionando, o método pode ser removido de `Customer`; porém, ele é chamado em `statement`:

```java
thisAmount = amountFor(each);
```

Logo, essa chamada deve ser atualizada para:

```java
thisAmount = each.getCharge();
```

**Commit & Push**


# Refactoring 4: Replace Temp with Query

Esse refactoring substitui uma variável local e temporária (temp) por uma chamada de função (query). No caso, vamos substituir toda referência a `thisAmount` por uma chamada a `each.getCharge()`. Veja o código após o refactoring:

```java
public String statement() {
   double totalAmount = 0;
   int frequentRenterPoints = 0;
   Enumeration rentals = _rentals.elements();
   String result = "Rental Record for " + getName() + "\n";
   while (rentals.hasMoreElements()) {
      Rental each = (Rental) rentals.nextElement();

      // add frequent renter points
      frequentRenterPoints ++;
      // add bonus for a two day new release rental
      if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) &&
         each.getDaysRented() > 1) frequentRenterPoints ++;

      // show figures for this rental
      result += "\t" + each.getMovie().getTitle()+ "\t" + String.valueOf
         (each.getCharge()) + "\n";
      totalAmount += each.getCharge();

   }
   
   // add footer lines
   result +=  "Amount owed is " + String.valueOf(totalAmount) + "\n";
   result += "You earned " + String.valueOf(frequentRenterPoints)
              + " frequent renter points";
   return result;
}
```

Resumindo o que foi feito acima: `thisAmount` sumiu e, nos dois pontos em que era usada, substitui-se por uma chamada a `getCharge()`.

Motivação para esse refactoring (chamado Replace Temp with Query): ficar livre de variáveis temporárias, que tendem a dificultar o entendimento do código; pois você tem que lembrar o que elas armazenam. Claro, pode-se alegar que isso causa um problema de performance. Porém, esse possível problema pode ser inclusive resolvido pelo compilador (isto é, pelas estratégias de otimização de código implementadas pelo compilador de Java).


**Commit & Push**

# Refactoring 5: Extract Method

Vamos decompor mais uma vez `statement`, para ir diminuindo seu tamanho e complexidade. Para isso, vamos extrair um método, chamado `getFrequentRenterPoints` com o código relativo ao comentário *add frequent renter points*.

Veja como deve ficar o código após o refactoring:


```java
class Customer...
   public String statement() {
      double totalAmount = 0;
      int frequentRenterPoints = 0;
      Enumeration rentals = _rentals.elements();
      String result = "Rental Record for " + getName() + "\n";
      while (rentals.hasMoreElements()) {
         Rental each = (Rental) rentals.nextElement();
         
         frequentRenterPoints += each.getFrequentRenterPoints();

         // show figures for this rental
         result += "\t" + each.getMovie().getTitle()+ "\t" +
                String.valueOf(each.getCharge()) + "\n";
         totalAmount += each.getCharge();
      }

      // add footer lines
      result +=  "Amount owed is " + String.valueOf(totalAmount) + "\n";
      result += "You earned " + String.valueOf(frequentRenterPoints) +
             " frequent renter points";
      return result;
  }

class Rental...
   int getFrequentRenterPoints() {
       if ((getMovie().getPriceCode() == Movie.NEW_RELEASE) && getDaysRented() > 1)
          return 2;
       else
          return 1;
   }
```

**Commit & Push**

# Refactoring 6: Replace Temp With Query

Mais duas variáveis locais (temp) vão ser extraídas para funções (queries). São elas:

* `totalAmount` vai ser substituída por `getTotalCharge()`
* `frequentRenterPoints` vai ser substituída por `getTotalFrequentRenterPoints`.

Veja como deve ficar o código após esses dois refactorings:


```java
public String statement() {
   Enumeration rentals = _rentals.elements();
   String result = "Rental Record for " + getName() + "\n";
   while (rentals.hasMoreElements()) {
      Rental each = (Rental) rentals.nextElement();

      // show figures for this rental
      result += "\t" + each.getMovie().getTitle()+ "\t" +
                String.valueOf(each.getCharge()) + "\n";
   }

   // add footer lines
   result +=  "Amount owed is " + String.valueOf(getTotalCharge()) + "\n";
   result += "You earned " + String.valueOf(getTotalFrequentRenterPoints()) +
                   " frequent renter points";
   return result;
}
    
    private double getTotalCharge() {
       double result = 0;
       Enumeration rentals = _rentals.elements();
       while (rentals.hasMoreElements()) {
          Rental each = (Rental) rentals.nextElement();
          result += each.getCharge();
       }
       return result;
     }

     private int getTotalFrequentRenterPoints(){
        int result = 0;
        Enumeration rentals = _rentals.elements();
        while (rentals.hasMoreElements()) {
           Rental each = (Rental) rentals.nextElement();
           result += each.getFrequentRenterPoints();
        }
        return result;
     }
```

Dois comentários breve, sobre alguns pontos que você já pode estar pensando sobre os últimos refactorings:

* Eles aumentaram o tamanho do código: porém, também não foi tanto assim ...
* Eles fizeram com o que o loop de `rentals` seja percorrido três vezes; na primeira versão do código, esse loop era executado uma única vez. Isso vai gerar problemas de performance? Talvez sim; mas, provavelmente na maioria dos casos, não vai fazer diferença, pois um cliente não tem tantos filmes alugados.


**Commit & Push**

# Nova feature: Statement em HTML

Neste passo, não vamos refatorar, mas introduzir uma nova feature: imprimir o comprovante de aluguel em HTML.

Para isso, vamos criar um novo método, chamado `htmlstatement`:

```java
public String htmlStatement() {
   Enumeration rentals = _rentals.elements();
   String result = "<H1>Rentals for <EM>" + getName() + "</EM></H1><P>\n";
   while (rentals.hasMoreElements()) {
      Rental each = (Rental) rentals.nextElement();
      // show figures for each rental
      result += each.getMovie().getTitle()+ ": " +
                String.valueOf(each.getCharge()) + "<BR>\n";
   }
   
   // add footer lines
   result +=  "<P>You owe <EM>" + String.valueOf(getTotalCharge()) + "</EM><P>\n";
   result += "On this rental you earned <EM>" +
          String.valueOf(getTotalFrequentRenterPoints()) +
          "</EM> frequent renter points<P>";
   return result;
}
```

Vantagem: conseguimos reusar todos os métodos criados anteriormente, incluindo:  `getCharge()`, `getTotalCharge()` e `getTotalFrequentRenterPoints`. Por isso, a criação do novo método foi bem rápida e não causou duplicação de código (ou uma duplicação pequena, assumindo que ainda existe alguma lógica repetida, com o método `statement`).


**Commit & Push**

# Refactoring 7: Replace Conditional with Polymorphism

Primeiro, não faz sentido ter um switch que depende de um atributo (`_priceCode`) de uma outra classe (`Movie)`, como em:

```java
class Rental...
   double getCharge() {
      double result = 0;
      switch (getMovie().getPriceCode()) {
         case Movie.REGULAR:
            result += 2;
             if (getDaysRented() > 2)
                result += (getDaysRented() - 2) * 1.5;
             break;
         case Movie.NEW_RELEASE:
            result += getDaysRented() * 3;
            break;
         case Movie.CHILDRENS:
            result += 1.5;
            if (getDaysRented() > 3)
               result += (getDaysRented() - 3) * 1.5;
            break;
       }
       return result;
  }
```

Logo, vamos extrair esse switch para um método e depois movê-lo para a class `Movie`:

```java
class Movie...
   double getCharge(int daysRented) {
      double result = 0;
      switch (getPriceCode()) {
         case Movie.REGULAR:
            result += 2;
            if (daysRented > 2)
               result += (daysRented - 2) * 1.5;
            break;
         case Movie.NEW_RELEASE:
            result += daysRented * 3;
            break;
         case Movie.CHILDRENS:
            result += 1.5;
            if (daysRented > 3)
               result += (daysRented - 3) * 1.5;
            break;
      }
      return result;
   }
```

O método antigo agora apenas inclui uma chamada para o método novo:

```java
class Rental...
   double getCharge() {
      return _movie.getCharge(_daysRented);
   }
```

Vamos agora também mover `getFrequentRenterPoints` para `Movie`; ou seja, é melhor que métodos que usam informações sobre tipos de filme estejam todos na classe `Movie`:

```java
class Rental...
   int getFrequentRenterPoints() {
       return _movie.getFrequentRenterPoints(_daysRented);
   }


class Movie...
   int getFrequentRenterPoints(int daysRented) {
       if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1)
          return 2;
       else
          return 1;
}
```

Por fim, herança, como no diagrama abaixo (**errata**: existe um erro no diagrama, que está no livro; onde consta `getCharge`, leia-se `getPriceCode`).


![heranca](https://github.com/mtov/AulaPraticaRefactoring/blob/master/classdiagram.png)


Isto é:

```java     
abstract class Price {
   abstract int getPriceCode();
}
 
class ChildrensPrice extends Price {
   int getPriceCode() {
       return Movie.CHILDRENS;
   }
}
 
class NewReleasePrice extends Price {
   int getPriceCode() {
       return Movie.NEW_RELEASE;
   }
}
 
class RegularPrice extends Price {
   int getPriceCode() {
       return Movie.REGULAR;
   }
}
```

Agora, em `Movie`, vamos:

* remover o campo `_priceCode` 
* criar um campo `_price` do tipo `Price`
* alterar o construtor, para chamar `_price.setPriceCode`
* criar um métodos `getPriceCode` e `setPriceCode`:
* remover o campo `_priceCode` criar um campo `_price` do tipo `Price`), alterar o construtor e criar um métodos `getPriceCode` e `setPriceCode`:

```java
class Movie...

   private Price _price;

   public Movie(String name, int priceCode) {
      _title = name;
      setPriceCode(priceCode);
   }
    
   public int getPriceCode() {
      return _price.getPriceCode();
   }
   
   public void setPriceCode(int arg) {
      switch (arg) {
         case REGULAR:
            _price = new RegularPrice();
            break;
         case CHILDRENS:
            _price = new ChildrensPrice();
            break;
         case NEW_RELEASE:
            _price = new NewReleasePrice();
            break;
         default:
            throw new IllegalArgumentException("Incorrect Price Code");
      }
   }
   
```

Mais um refactoring, agora mover `getCharge` de `Movie` para `Price`:

```java
class Movie...
   double getCharge(int daysRented) {
      return _price.getCharge(daysRented);
   }

  class Price...
     double getCharge(int daysRented) {
        double result = 0;
        switch (getPriceCode()) {
           case Movie.REGULAR:
                result += 2;
                if (daysRented > 2)
                   result += (daysRented - 2) * 1.5;
                break;
           case Movie.NEW_RELEASE:
                result += daysRented * 3;
                break;
           case Movie.CHILDRENS:
                result += 1.5;
                if (daysRented > 3)
                    result += (daysRented - 3) * 1.5;
                break;
        }
        return result;
    }
```

Caminhando para o final, vamos decompor `getCharge`, criando métodos específicos nas subclasses de `Price` (veja que na classe `Price`, propriamente dita, `getCharge` vai ficar como um método abstrato):

```java
class Price...
   abstract double getCharge(int daysRented);
     
class RegularPrice ...
   double getCharge(int daysRented) {
      double result = 2;
      if (daysRented > 2)
         result += (daysRented - 2) * 1.5;
      return result;
   }
     
class ChildrensPrice ...
    double getCharge(int daysRented) {
       double result = 1.5;
       if (daysRented > 3)
          result += (daysRented - 3) * 1.5;
       return result;
     }
     
class NewReleasePrice ...
    double getCharge(int daysRented){
       return daysRented * 3;
    } 
```

E agora vamos fazer algo bem parecido com o método `getFrequentRenterPoints`. 

Para isso, como um primeiro passo, ainda intermediário, vamos mover esse método de `Movie` para `Price`:
  
```java  
class Movie ...
   int getFrequentRenterPoints(int daysRented) {
         return _price.getFrequentRenterPoints(daysRented);
   }
   
 class Price...
   int getFrequentRenterPoints(int daysRented) {
       if ((getPriceCode() == Movie.NEW_RELEASE) && daysRented > 1)
          return 2;
       else
          return 1;
   }
```  

E agora vamos decompor `getFrequentRenterPoints`; ele vai ficar com uma versão "genérica e concreta" em `Price` e, outra, para tratar um caso especial, em `NewReleasePrice`:

```java  
class Price...
   int getFrequentRenterPoints(int daysRented) {
       return 1;
   }

class NewReleasePrice
   int getFrequentRenterPoints(int daysRented) {
       return (daysRented > 1) ? 2: 1;
   }
``` 

Pronto, com isso terminamos: **Commit & Push**

# Comentário Final

Para terminar mesmo, leia e reflita com calma sobre os comentários finais do Fowler (ele argumenta sobre as vantagens do último refactoring):

> Putting in the state pattern was quite an effort. Was it worth it? The gain is that if I change any of price’s behavior, add new prices, or add extra price-dependent behavior, the change will be much easier to make. The rest of the application does not know about the use of the state pattern. For the tiny amount of behavior I currently have, it is not a big deal. In a more complex system with a dozen or so price-dependent methods, this would make a big difference. All these changes were small steps. It seems slow to write it this way, but not once did I have to open the debugger, so the process actually flowed quite quickly. It took me much longer to write this section of the book than it did to change the code.

> I’ve now completed the second major refactoring. It is going to be much easier to change the classification structure of movies, and to alter the rules for charging and the frequent renter point system

E também a seção que finaliza o capítulo (**Final Thoughts**):

> This is a simple example, yet I hope it gives you the feeling of what refactoring is like. I’ve used several refactorings, including Extract Method (110), Move Method (142), and Replace Conditional with Polymorphism (255). All these lead to better-distributed responsibilities and code that is easier to maintain. It does look rather different from procedural style code, and that takes some getting used to. But once you are used to it, it is hard to go back to procedural programs.

The most important lesson from this example is the rhythm of refactoring: test, small change, test, small change, test, small change. It is that rhythm that allows refactoring to move quickly and safely.

