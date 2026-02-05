O [[Lombok]] cria automaticamente os `@Getter / @Setter` usando anotações da biblioteca. sem ele teria que configurar a mão.

```java
public class User {  
	private String firstName;  
	private String lastName;  
	  
	public String getFirstName() {  
	return firstName;  
	}  
	  
	public void setFirstName(String firstName) {  
	this.firstName = firstName;  
	}  
	  
	public String getLastName() {  
	return lastName;  
	}  
	  
	public void setLastName(String lastName) {  
	this.lastName = lastName;  
	}  
}
```

Porém com o [[Lombok]] posso ter resultado simplificado dessa forma

```java
import lombok.Getter;  
import lombok.Setter;  
  
@Getter  
@Setter  
public class User {  
	private String firstName;  
	private String lastName;  
}
```

Dessa foram ele gerará automaticamente os valores de nomes. Podendo também controlar a [[Visibilidade de Classes]]

```java
@Getter(AccessLevel.PUBLIC)  
@Setter(AccessLevel.PRIVATE)  
public class User {  
	private String firstName;  
	private String lastName;  
}
```

Mantendo ela `Publica` ou `Privada` 