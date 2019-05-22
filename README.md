# AulaPraticaRefactoring
Aula prática sobre refactoring (usando exemplo inicial do Livro do Fowler)

# Descrição e Versão Inicial

As classes que vamos usar fazem parte de um sistema de video-locadora, para aluguel de vídeos.

Inicialmente, são três classes:

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
  ```
```


