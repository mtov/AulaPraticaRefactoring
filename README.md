# Aula Prática sobre Refactoring

Aula prática sobre refactoring, usando exemplo inicial do Livro do Fowler.

Objetivo: realizar alguns refactorings em um sistema hipotético.

Instruções:

* Primeiro, crie um repositório no GitHub.

* Vá seguindo o roteiro, refactoring a refactoring.

* Após cada refactoring, dê um **COMMIT & PUSH**. Esses commits serão usados na correção, para garantir que realizou todos os refactorings solicitados.


# Descrição e Versão Inicial

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
* `frequentRenterPoints` vai ser substituída por ``.

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

**Commit & Push**

