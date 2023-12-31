# Esempio Spring Boot Data - inizio CRUD
## Fase 1: dipendenze e indirizzi
### MAMP
Far partire MAMP o PHP Launcher, in particolare il servizio riguardante il database (MySQL)

### DBEaver
Creare il database e annotare il nome in DBEaver

### `POM.XML`
Includere le dipendenze nel `pom.xml` per accedere al **database**:
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>8.0.32</version>
</dependency>
```
#### MariaDB
Se dovessi utilizzare MariaDB e nel caso in cui `MariaDB` desse problemi, aggiungere `mariadb-java-client` come dipendenza
```xml
<dependency>
	<groupId>org.mariadb.jdbc</groupId>
	<artifactId>mariadb-java-client</artifactId>
</dependency>
```
### `application.properties`
Definire le seguenti impostazioni nel file di configurazione `application.properties` avendo cura di **MODIFICARE** il **nome del database**, **username** e **password** :
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/nome_db
spring.datasource.username=root
spring.datasource.password=

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=create-drop
```

#### MariaDB
In caso in cui `MariaDB` desse problemi sostituire il `dialect` con la seguente riga
```properties
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
```
### Risultato atteso
In console non dovrebbero apparire errori, e dovrebbero essere visibili dei log riguardanti il boot della connessione con il database
![Alt text](image.png)

## Fase 2: Model
Generare una class in un package dedicato (es: `....db.pojo`) con annotation a livello di classe `@Entity`, la chiave primaria e le variabili di istanza come mappa per le colonne della tabella in db
#### Code
```java
import java.time.LocalDate;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Book {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int id;
	
	@Column(length = 128, nullable = false)
	private String title;
	private String subTitle;
	
	private LocalDate releaseDate;
	@Column(unique = true)
	private String isbn;
	
	public Book() { }
	public Book(String title, String subTitle, LocalDate releaseDate, String isbn) {
	
		setTitle(title);
		setSubTitle(subTitle);
		setReleaseDate(releaseDate);
		setIsbn(isbn);
	}
	
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	public String getSubTitle() {
		return subTitle;
	}
	public void setSubTitle(String subTitle) {
		this.subTitle = subTitle;
	}
	public LocalDate getReleaseDate() {
		return releaseDate;
	}
	public void setReleaseDate(LocalDate releaseDate) {
		this.releaseDate = releaseDate;
	}
	public String getIsbn() {
		return isbn;
	}
	public void setIsbn(String isbn) {
		this.isbn = isbn;
	}
	
	@Override
	public String toString() {
		
		return "[" + getId() + "] Book:\n"
					+ "title: " + getTitle() + "\n"
					+ "sub-title: " + getSubTitle() + "\n"
					+ "release date: " + getReleaseDate() + "\n"
					+ "ISBN: " + getIsbn();
	}
}
```
#### Risultato
Dopo aver creato la classe correttamente e' possibile compilare nuovamente il progetto e verificare in **DBEaver** che la tabella sia stata creata correttamente in db
![Alt text](image-1.png)

## Fase 3: Insert
#### Repository
Generare l'interfaccia **repository** che estendera' `JpaRepository<E, T>` passandogli come parametri **generics** (tra le parentesi angolari) prima la classe **model**, poi il relativo tipo di dato della chiave primaria (quasi sempre `Integer`)
```java
import org.java.app.db.pojo.Book;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface BookRepo extends JpaRepository<Book, Integer> {

}
```

#### Service
Generare la classe **service** per esporre i metodi del **repository**. Dopo aver importato il **repository** tramite `@Autowire` definire almeno i 3 metodi principali per interfacciarsi con una tabella in db (**insert**, **findAll**, **findById**)
```java
@Service
public class BookService {

	@Autowired
	private BookRepo bookRepo;
	
	public void save(Book book) {
		
		bookRepo.save(book);
	}
	public List<Book> findAll() {
		
		return bookRepo.findAll();
	}
	public Book findById(int id) {
		
		return bookRepo.findById(id).get();
	}
}
```

#### CommandLineRunner
All'interno dello `@SpringBootApplication` (il file `java` principale) implementare l'intefaccia `CommandLineRunner` e il relativo metodo `void run(String... args) throw Exception` in cui verranno fatti alcuni inserimenti di prova nella tabella in db
```java
import java.time.LocalDate;
import java.util.List;

import org.java.app.db.pojo.Book;
import org.java.app.db.serv.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application implements CommandLineRunner {

	@Autowired
	private BookService bookService;
	
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

	@Override
	public void run(String... args) throws Exception {
		
		Book book1 = new Book("mio titolo 1", "mio sottotitolo 1", LocalDate.now(), "1234234");
		Book book2 = new Book("mio titolo 2", "mio sottotitolo 2", LocalDate.now(), "1234235");
		Book book3 = new Book("mio titolo 3", "mio sottotitolo 3", LocalDate.now(), "1234236");
		
		bookService.save(book1);
		bookService.save(book2);
		bookService.save(book3);
		
		System.out.println("Insert OK!");
	}
}
```

#### Risultato
Dopo aver ri-compilato il progetto, sara' possibile vedere i dati salvati in **persistenza** all'interno del db tramite **DBEaver**
![Alt text](image-2.png)

## Fase 4: MVC
#### Controller
All'interno del controller e' importare il `service` dei `Book` tramite `@Autowired`. Aggiungere poi almeno 2 rotte per le funzione di **show** e **index**. Recuperare i dati dal db tramite il `service` e mandarli al **front-end** tramite il `model`
```java
import java.util.List;

import org.java.app.db.pojo.Book;
import org.java.app.db.serv.BookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/books")
public class BookController {

	@Autowired
	private BookService bookService;
	
	@GetMapping
	public String getIndex(Model model) {
		
		List<Book> books = bookService.findAll();
		model.addAttribute("books", books);
		
		return "book-index";
	}
	
	@GetMapping("/{id}")
	public String getShow(@PathVariable int id, Model model) {
		
		Book book = bookService.findById(id);
		model.addAttribute("book", book);
		
		return "book-show";
	}
}
```

#### Thymeleaf
Definire i due file `thymeleaf` per le due diverse rotte, che andranno a utilizzare **liste** e **model** per rappresentare i dati
##### `book-index.html`
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1>Books:</h1>
	<ul>
		<li
			th:each="book : ${books}"
			th:object="${book}"
		>
			<a th:href="@{/books/{id}(id=*{id})}">
				[[*{isbn}]] - [[*{title}]]	
			</a>
		</li>
	</ul>
</body>
</html>
```
##### `book-show.html`
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body th:object="${book}">
	[[*{isbn}]] - [[*{title}]] <br />
	<small>[[*{subTitle}]]</small> <br />	
	[[*{releaseDate}]]		
</body>
</html>
```

#### Risultato
E' ora possibile accedere alla rotta definita nel controller (`/books`) per vedere la lista di libri presenti in db con il relativo collegamento ai dettagli del singolo libro
##### `localhost:8080/books`
![Alt text](image-3.png)

##### `localhost:8080/books/1`
![Alt text](image-4.png)