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

Inicialmente, mova esse método para `Rental`, mas com o nome `getCharge()`; a versão antiga vai ser alterada para apenas delegar a chamada, para o método movido:

```
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

class Customer...
   private double amountFor(Rental aRental) {
      return aRental.getCharge();  // agora apenas delega chamada para método movido
   }

```



