# Hibernate Basics

> _Click &#9733; if you like the project. Your contributions are heartily ♡ welcome._

<br/>

## Q. How to integrate hibernate with spring boot?

**Step 01: Maven Dependencies:**

```xml
<!-- pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.javaexample.demo</groupId>
    <artifactId>SpringBoot2Demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBoot2Demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

- **spring-boot-starter-data-jpa**(required): It includes spring data, hibernate, HikariCP, JPA API, JPA Implementation (default is hibernate), JDBC and other required libraries.

**Step 02: Create JPA entity classes:**

```java
/** EmployeeEntity.java **/

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="TBL_EMPLOYEES")
public class EmployeeEntity {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name="first_name")
    private String firstName;

    @Column(name="last_name")
    private String lastName;

    @Column(name="email", nullable=false, length=200)
    private String email;

    //Setters and getters left out for brevity.

    @Override
    public String toString() {
        return "EmployeeEntity [id=" + id + ", firstName=" + firstName +
                ", lastName=" + lastName + ", email=" + email   + "]";
    }
}
```

**Step 03: Create JPA Repository:**

Extend `JpaRepository` interface to allows to create repository implementations automatically, at runtime, for any given entity class. The types of entity class and it’s ID field are specified in the generic parameters on JpaRepository.

```java
/** EmployeeRepository.java **/
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import com.javaexample.demo.entity.EmployeeEntity;

@Repository
public interface EmployeeRepository extends JpaRepository<EmployeeEntity, Long> {

}
```

By this simple extension, EmployeeRepository inherits several methods for working with Employee persistence, including methods for saving, deleting, and finding Employee entities.

**Step 04: Properties Configuration:**

**application.properties:**

```properties
spring.datasource.url=jdbc:h2:file:~/test
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# Enabling H2 Console
spring.h2.console.enabled=true

# Custom H2 Console URL
spring.h2.console.path=/h2-console


#Turn Statistics on and log SQL stmts

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

#If want to see very extensive logging
spring.jpa.properties.hibernate.generate_statistics=true
logging.level.org.hibernate.type=trace
logging.level.org.hibernate.stat=debug

#Schema will be created using schema.sql and data.sql files

spring.jpa.hibernate.ddl-auto=none
```

```sql
## schama.sql
DROP TABLE IF EXISTS TBL_EMPLOYEES;

CREATE TABLE TBL_EMPLOYEES (
    id INT AUTO_INCREMENT  PRIMARY KEY,
    first_name VARCHAR(250) NOT NULL,
    last_name VARCHAR(250) NOT NULL,
    email VARCHAR(250) DEFAULT NULL
);

## data.sql
INSERT INTO TBL_EMPLOYEES
    (first_name, last_name, email)
VALUES
    ('Lokesh', 'Gupta', 'abc@gmail.com'),
    ('Deja', 'Vu', 'xyz@email.com'),
    ('Caption', 'America', 'cap@marvel.com');
```

**Step 05. Spring boot hibernate demo:**

To test hibernate configuration with Spring boot, we need to autowire the EmployeeRepository dependency in a class and use it’s method to save or fetch employee entities.

**SpringBoot2DemoApplication.java:**

```java
import java.util.Optional;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import com.javaexample.demo.entity.EmployeeEntity;
import com.javaexample.demo.repository.EmployeeRepository;

@SpringBootApplication
public class SpringBoot2DemoApplication implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    EmployeeRepository repository;

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot2DemoApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception
    {
        Optional<EmployeeEntity> emp = repository.findById(2L);

        logger.info("Employee id 2 -> {}", emp.get());
    }
}
```

Output

```java
Tomcat initialized with port(s): 8080 (http)
Starting service [Tomcat]
Starting Servlet engine: [Apache Tomcat/9.0.19]
Initializing Spring embedded WebApplicationContext
Root WebApplicationContext: initialization completed in 5748 ms

HikariPool-1 - Starting...
HikariPool-1 - Start completed.
HHH000204: Processing PersistenceUnitInfo [
    name: default
    ...]
HHH000412: Hibernate Core {5.3.10.Final}
HHH000206: hibernate.properties not found
HCANN000001: Hibernate Commons Annotations {5.0.4.Final}
HHH000400: Using dialect: org.hibernate.dialect.H2Dialect

Initialized JPA EntityManagerFactory for persistence unit 'default'
Initializing ExecutorService 'applicationTaskExecutor'
spring.jpa.open-in-view is enabled by default. Therefore, database queries may be performed during view rendering.
Explicitly configure spring.jpa.open-in-view to disable this warning
Tomcat started on port(s): 8080 (http) with context path ''
Started SpringBoot2DemoApplication in 17.638 seconds (JVM running for 19.1)

Hibernate:
    select
        employeeen0_.id as id1_0_0_,
        employeeen0_.email as email2_0_0_,
        employeeen0_.first_name as first_na3_0_0_,
        employeeen0_.last_name as last_nam4_0_0_
    from
        tbl_employees employeeen0_
    where
        employeeen0_.id=?

Employee id 2 -> EmployeeEntity [id=2, firstName=Deja, lastName=Vu, email=xyz@email.com]
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is Hibernate Validator Framework?

**Answer:** Hibernate Validator is the reference implementation of the Bean Validation specification (JSR 303 and JSR 349). It provides a framework for validating Java objects using annotations, ensuring data integrity and consistency before data is persisted to the database. It integrates seamlessly with Hibernate and JPA, allowing automatic validation during entity lifecycle events like save, update, and delete.

**Key Features:**

- Declarative validation using annotations like `@NotNull`, `@Size`, `@Email`.
- Custom validation constraints.
- Integration with Hibernate for automatic validation.
- Support for validation groups and custom validators.

**Example:**

```java
@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;

    @NotNull
    @Size(min = 2, max = 50)
    private String name;

    @Email
    private String email;

    // Getters and setters
}
```

When saving a `User` entity, Hibernate Validator will automatically check the constraints.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is the benefit of Hibernate Tools Eclipse plugin?

**Answer:** Hibernate Tools is an Eclipse plugin that provides a set of tools to simplify Hibernate development within the Eclipse IDE. It offers features for code generation, reverse engineering, and query editing, making it easier to work with Hibernate mappings and entities.

**Key Benefits:**

- **Code Generation:** Automatically generates entity classes, DAOs, and mapping files from database schemas.
- **Reverse Engineering:** Creates Hibernate mappings and entities from existing databases.
- **HQL Editor:** Provides a dedicated editor for writing and testing HQL queries with syntax highlighting and validation.
- **Mapping File Editor:** Assists in editing XML mapping files with validation and auto-completion.
- **Console Configuration:** Allows configuration of Hibernate consoles for testing queries and mappings.
- **Integration with Eclipse:** Seamless integration for faster development, debugging, and deployment.

**Example Usage:**

- Use the "Hibernate Code Generation" wizard to generate entities from a database table.
- Open the HQL Editor to write and execute queries like `from User where name like 'A%'`.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is HibernateTemplate class?

**Answer:** HibernateTemplate is a helper class provided by the Spring Framework to simplify the integration and usage of Hibernate in Spring-based applications. It acts as a wrapper around the Hibernate Session, providing template methods for common database operations such as save, update, delete, and query. HibernateTemplate handles the lifecycle of Hibernate sessions and transactions automatically, reducing boilerplate code and potential errors. It integrates with Spring's transaction management and exception translation, converting Hibernate exceptions into Spring's DataAccessException hierarchy.

**Key Features:**

- Manages Hibernate Session lifecycle.
- Provides methods for CRUD operations.
- Handles transactions and exceptions.
- Supports both Hibernate 3 and Hibernate 4/5.

**Example Usage:**

```java
import org.springframework.orm.hibernate5.HibernateTemplate;
import org.springframework.stereotype.Repository;

@Repository
public class EmployeeDao {

    @Autowired
    private HibernateTemplate hibernateTemplate;

    public void saveEmployee(Employee employee) {
        hibernateTemplate.save(employee);
    }

    public Employee getEmployee(int id) {
        return hibernateTemplate.get(Employee.class, id);
    }

    public List<Employee> getAllEmployees() {
        return hibernateTemplate.loadAll(Employee.class);
    }

    public void updateEmployee(Employee employee) {
        hibernateTemplate.update(employee);
    }

    public void deleteEmployee(Employee employee) {
        hibernateTemplate.delete(employee);
    }
}
```

In this example, HibernateTemplate is injected into a DAO class, and methods like `save()`, `get()`, and `loadAll()` are used to perform database operations without manually managing sessions or transactions.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How to integrate Hibernate with Servlet or Struts2 web applications?

**Answer:** Integrating Hibernate with Servlet or Struts2 web applications involves setting up Hibernate configuration, managing sessions, and performing database operations within the web layer. Below are the steps and examples for both Servlet and Struts2.

### General Steps:

1. **Add Dependencies:** Include Hibernate JARs (e.g., hibernate-core, hibernate-entitymanager) and database driver in your project (e.g., via Maven or lib folder).
2. **Create Hibernate Configuration:** Set up `hibernate.cfg.xml` with database properties, dialect, and entity mappings.
3. **Create Entity Classes:** Annotate POJOs with Hibernate annotations like `@Entity`, `@Id`, etc.
4. **Set Up SessionFactory:** Create a utility class to build and provide SessionFactory.
5. **Manage Sessions in Web Layer:** In servlets or actions, obtain sessions, perform operations, and handle transactions.
6. **Handle Transactions:** Use Hibernate transactions for data consistency.
7. **Close Resources:** Always close sessions to avoid leaks.

### Example with Servlet:

**HibernateUtil.java (SessionFactory Setup):**

```java
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {
    private static final SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory() {
        try {
            return new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();
        } catch (Throwable ex) {
            System.err.println("Initial SessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        getSessionFactory().close();
    }
}
```

**EmployeeServlet.java:**

```java
import java.io.IOException;
import java.util.List;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.hibernate.Session;
import org.hibernate.Transaction;

public class EmployeeServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = null;
        try {
            tx = session.beginTransaction();
            List<Employee> employees = session.createQuery("from Employee").list();
            request.setAttribute("employees", employees);
            tx.commit();
        } catch (Exception e) {
            if (tx != null) tx.rollback();
            e.printStackTrace();
        } finally {
            session.close();
        }
        request.getRequestDispatcher("employees.jsp").forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String name = request.getParameter("name");
        Employee emp = new Employee();
        emp.setName(name);

        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = null;
        try {
            tx = session.beginTransaction();
            session.save(emp);
            tx.commit();
        } catch (Exception e) {
            if (tx != null) tx.rollback();
            e.printStackTrace();
        } finally {
            session.close();
        }
        response.sendRedirect("employees");
    }
}
```

### Example with Struts2:

**EmployeeAction.java:**

```java
import com.opensymphony.xwork2.ActionSupport;
import org.hibernate.Session;
import org.hibernate.Transaction;
import java.util.List;

public class EmployeeAction extends ActionSupport {
    private List<Employee> employees;
    private Employee employee;

    public String list() {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = null;
        try {
            tx = session.beginTransaction();
            employees = session.createQuery("from Employee").list();
            tx.commit();
        } catch (Exception e) {
            if (tx != null) tx.rollback();
            e.printStackTrace();
        } finally {
            session.close();
        }
        return SUCCESS;
    }

    public String save() {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = null;
        try {
            tx = session.beginTransaction();
            session.save(employee);
            tx.commit();
        } catch (Exception e) {
            if (tx != null) tx.rollback();
            e.printStackTrace();
        } finally {
            session.close();
        }
        return SUCCESS;
    }

    // Getters and setters
    public List<Employee> getEmployees() {
        return employees;
    }

    public void setEmployees(List<Employee> employees) {
        this.employees = employees;
    }

    public Employee getEmployee() {
        return employee;
    }

    public void setEmployee(Employee employee) {
        this.employee = employee;
    }
}
```

**struts.xml:**

```xml
<struts>
    <package name="default" extends="struts-default">
        <action name="listEmployees" class="com.example.EmployeeAction" method="list">
            <result name="success">employees.jsp</result>
        </action>
        <action name="saveEmployee" class="com.example.EmployeeAction" method="save">
            <result name="success" type="redirect">listEmployees</result>
        </action>
    </package>
</struts>
```

**Best Practices:**

- Use a session-per-request pattern or Open Session in View filter to manage sessions.
- Handle exceptions properly and log them.
- For production, consider using connection pooling (e.g., C3P0 with Hibernate).
- Integrate with Spring for better transaction management if possible.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Mention the differences between JPA and Hibernate?

JPA is a specification for accessing, persisting and managing the data between Java objects and the relational database.

Where as, Hibernate is the actual implementation of JPA guidelines. When hibernate implements the JPA specification, this will be certified by the JPA group upon following all the standards mentioned in the specification. For example, JPA guidelines would provide information of mandatory and optional features to be implemented as part of the JPA implementation.

| Hibernate                                                                                           | JPA                                                                                     |
| --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Hibernate is the object-relational mapping framework which helps to deal with the data persistence. | It is the Java specification to manage the java application with relational data.       |
| It’s is one of the best JPA providers.                                                              | It is the only specification which doesn’t deal with any implementation.                |
| In this, we use Session for handling the persistence in an application.                             | In this, we use the Entity manager.                                                     |
| It is used to map Java data types with database tables and SQL data types.                          | It is the standard API which allows developers to perform database operations smoothly. |
| The Query language in this is Hibernate Query Language.                                             | The query language of JPA is JPQL (Java Persistence Query Language)                     |

## Q. What is HQL and what are its benefits?

Hibernate Query Language (HQL) is an object-oriented query language, similar to SQL, but instead of operating on tables and columns, HQL works with persistent objects and their properties. HQL queries are translated by Hibernate into conventional SQL queries, which in turns perform action on database.

**Advantages Of HQL:**

- HQL is database independent, means if we write any program using HQL commands then our program will be able to execute in all the databases with out doing any further changes to it
- HQL supports object oriented features like Inheritance, polymorphism,Associations(Relation ships)
- HQL is initially given for selecting object from database and in hibernate 3.x we can doDML operations ( insert, update…) too

**Different Ways Of Construction HQL Select:**

**FROM Clause:**

```sql
/** In SQL **/
sql> select * from Employee

/** In HQL **/
String hql = "FROM Employee";
Query query = session.createQuery(hql);
List results = query.list();
```

**AS Clause:**

```sql
String hql = "FROM Employee AS E";
Query query = session.createQuery(hql);
List results = query.list();
```

**SELECT Clause:**

```sql
String hql = "SELECT E.firstName FROM Employee E";
Query query = session.createQuery(hql);
List results = query.list();
```

**WHERE Clause:**

```sql
String hql = "FROM Employee E WHERE E.id = 10";
Query query = session.createQuery(hql);
List results = query.list();
```

**ORDER BY Clause:**

```sql
String hql = "FROM Employee E WHERE E.id > 10 ORDER BY E.salary DESC";
Query query = session.createQuery(hql);
List results = query.list();
```

**GROUP BY Clause:**

```sql
String hql = "SELECT SUM(E.salary), E.firtName FROM Employee E " +
             "GROUP BY E.firstName";
Query query = session.createQuery(hql);
List results = query.list();
```

**Using Named Parameters:**

Hibernate supports named parameters in its HQL queries. This makes writing HQL queries that accept input from the user easy and you do not have to defend against SQL injection attacks. Following is the simple syntax of using named parameters −

```sql
String hql = "FROM Employee E WHERE E.id = :employee_id";
Query query = session.createQuery(hql);
query.setParameter("employee_id",10);
List results = query.list();
```

**UPDATE Clause:**

```sql
String hql = "UPDATE Employee set salary = :salary "  +
             "WHERE id = :employee_id";
Query query = session.createQuery(hql);
query.setParameter("salary", 1000);
query.setParameter("employee_id", 10);
int result = query.executeUpdate();
System.out.println("Rows affected: " + result);
```

**DELETE Clause:**

```sql
String hql = "DELETE FROM Employee "  +
             "WHERE id = :employee_id";
Query query = session.createQuery(hql);
query.setParameter("employee_id", 10);
int result = query.executeUpdate();
System.out.println("Rows affected: " + result);
```

**INSERT Clause:**

```sql
String hql = "INSERT INTO Employee(firstName, lastName, salary)"  +
             "SELECT firstName, lastName, salary FROM old_employee";
Query query = session.createQuery(hql);
int result = query.executeUpdate();
System.out.println("Rows affected: " + result);
```

**Pagination using Query:**

```sql
String hql = "FROM Employee";
Query query = session.createQuery(hql);
query.setFirstResult(1);
query.setMaxResults(10);
List results = query.list();
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Mention the Key components of Hibernate?

**hibernate.cfg.xml**: This file has database connection details

**hbm.xml or Annotation**: Defines the database table mapping with POJO. Also defines the relation between tables in java way.  
**SessionFactory**:

- There will be a session factory per database.
- The SessionFacory is built once at start-up
- It is a thread safe class
- SessionFactory will create a new Session object when requested
  **Session**:
- The Session object will get physical connection to the database.
- Session is the Java object used for any DB operations.
- Session is not thread safe. Hence do not share hibernate session between threads
- Session represents unit of work with database
- Session should be closed once the task is completed

## Q. Explain Session object in Hibernate?

A Session is used to get a physical connection with a database. The Session object is lightweight and designed to be instantiated each time an interaction is needed with the database. Persistent objects are saved and retrieved through a Session object.

The lifecycle of a Session is bounded by the beginning and end of a logical transaction. The main function of the Session is to offer create, read and delete operations for instances of mapped entity classes. Instances may exist in one of three states:

- **transient** − A new instance of a persistent class, which is not associated with a Session and has no representation in the database and no identifier value is considered transient by Hibernate.

- **persistent** − You can make a transient instance persistent by associating it with a Session. A persistent instance has a representation in the database, an identifier value and is associated with a Session.

- **detached** − Once we close the Hibernate Session, the persistent instance will become a detached instance.

```java
Session session = factory.openSession();
Transaction tx = null;

try {
   tx = session.beginTransaction();
   // do some work
   ...
   tx.commit();
}

catch (Exception e) {
   if (tx!=null) tx.rollback();
   e.printStackTrace();
} finally {
   session.close();
}
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How transaction management works in Hibernate?

A **Transaction** is a sequence of operation which works as an atomic unit. A transaction only completes if all the operations completed successfully. A transaction has the Atomicity, Consistency, Isolation, and Durability properties (ACID).

In hibernate framework, **Transaction interface** that defines the unit of work. It maintains abstraction from the transaction implementation (JTA, JDBC). A transaction is associated with Session and instantiated by calling **session.beginTransaction()**.

| Transaction interface                           | Description                                                                                     |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------- |
| void begin()                                    | starts a new transaction.                                                                       |
| void commit()                                   | ends the unit of work unless we are in FlushMode.NEVER.                                         |
| void rollback()                                 | forces this transaction to rollback.                                                            |
| void setTimeout(int seconds)                    | It sets a transaction timeout for any transaction started by a subsequent call to begin on this | instance. |
| boolean isAlive()                               | checks if the transaction is still alive.                                                       |
| void registerSynchronization(Synchronization s) | registers a user synchronization callback for this transaction.                                 |
| boolean wasCommited()                           | checks if the transaction is commited successfully.                                             |
| boolean wasRolledBack()                         | checks if the transaction is rolledback successfully.                                           |

Example:

```java
Transaction transObj = null;
Session sessionObj = null;
try {
    sessionObj = HibernateUtil.buildSessionFactory().openSession();
    transObj = sessionObj.beginTransaction();

    //Perform Some Operation Here
    transObj.commit();
} catch (HibernateException exObj) {
    if(transObj!=null){
        transObj.rollback();
    }
    exObj.printStackTrace();
} finally {
    sessionObj.close();
}
```

## Q. Explain the Criteria object in Hibernate?

The Criteria API allows to build up a criteria query object programmatically; the `org.hibernate.Criteria` interface defines the available methods for one of these objects. The Hibernate Session interface contains several overloaded `createCriteria()` methods.

**1. Restrictions with Criteria:**

```java
Criteria cr = session.createCriteria(Employee.class);

// To get records having salary is equal to 2000
cr.add(Restrictions.eq("salary", 2000));
List results = cr.list();

// To get records having salary more than 2000
cr.add(Restrictions.gt("salary", 2000));

// To get records having salary less than 2000
cr.add(Restrictions.lt("salary", 2000));

// To get records having fistName starting with zara
cr.add(Restrictions.like("firstName", "zara%"));

// Case sensitive form of the above restriction.
cr.add(Restrictions.ilike("firstName", "zara%"));

// To get records having salary in between 1000 and 2000
cr.add(Restrictions.between("salary", 1000, 2000));

// To check if the given property is null
cr.add(Restrictions.isNull("salary"));

// To check if the given property is not null
cr.add(Restrictions.isNotNull("salary"));

// To check if the given property is empty
cr.add(Restrictions.isEmpty("salary"));

// To check if the given property is not empty
cr.add(Restrictions.isNotEmpty("salary"));
```

**2. Logical Expression Restrictions:**

```java
Criteria cr = session.createCriteria(Employee.class);

Criterion salary = Restrictions.gt("salary", 2000);
Criterion name = Restrictions.ilike("firstNname","zara%");

// To get records matching with OR conditions
LogicalExpression orExp = Restrictions.or(salary, name);
cr.add( orExp );

// To get records matching with AND conditions
LogicalExpression andExp = Restrictions.and(salary, name);
cr.add( andExp );

List results = cr.list();
```

**3. Pagination Using Criteria:**

```java
Criteria cr = session.createCriteria(Employee.class);
cr.setFirstResult(1);
cr.setMaxResults(10);
List results = cr.list();
```

**4. Sorting the Results:**

The Criteria API provides the **org.hibernate.criterion.Order** class to sort your result set in either ascending or descending order, according to one of your object's properties.

```java
Criteria cr = session.createCriteria(Employee.class);

// To get records having salary more than 2000
cr.add(Restrictions.gt("salary", 2000));

// To sort records in descening order
cr.addOrder(Order.desc("salary"));

// To sort records in ascending order
cr.addOrder(Order.asc("salary"));

List results = cr.list();
```

**5. Projections & Aggregations:**

The Criteria API provides the **org.hibernate.criterion.Projections** class, which can be used to get average, maximum, or minimum of the property values. The Projections class is similar to the Restrictions class, in that it provides several static factory methods for obtaining **Projection** instances.

```java
Criteria cr = session.createCriteria(Employee.class);

// To get total row count.
cr.setProjection(Projections.rowCount());

// To get average of a property.
cr.setProjection(Projections.avg("salary"));

// To get distinct count of a property.
cr.setProjection(Projections.countDistinct("firstName"));

// To get maximum of a property.
cr.setProjection(Projections.max("salary"));

// To get minimum of a property.
cr.setProjection(Projections.min("salary"));

// To get sum of a property.
cr.setProjection(Projections.sum("salary"));
```

**Criteria Queries Example:**

Employee.java

```java
public class Employee {
   private int id;
   private String firstName;
   private String lastName;
   private int salary;

   public Employee() {}

   public Employee(String fname, String lname, int salary) {
      this.firstName = fname;
      this.lastName = lname;
      this.salary = salary;
   }

   public int getId() {
      return id;
   }

   public void setId( int id ) {
      this.id = id;
   }

   public String getFirstName() {
      return firstName;
   }

   public void setFirstName( String first_name ) {
      this.firstName = first_name;
   }

   public String getLastName() {
      return lastName;
   }

   public void setLastName( String last_name ) {
      this.lastName = last_name;
   }

   public int getSalary() {
      return salary;
   }

   public void setSalary( int salary ) {
      this.salary = salary;
   }
}
```

EMPLOYEE.sql

```sql
create table EMPLOYEE (
   id INT NOT NULL auto_increment,
   first_name VARCHAR(20) default NULL,
   last_name  VARCHAR(20) default NULL,
   salary     INT  default NULL,
   PRIMARY KEY (id)
);
```

Hibernate Mapping File

```xml
<?xml version = "1.0" encoding = "utf-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
"-//Hibernate/Hibernate Mapping DTD//EN"
"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
   <class name = "Employee" table = "EMPLOYEE">

      <meta attribute = "class-description">
         This class contains the employee detail.
      </meta>

      <id name = "id" type = "int" column = "id">
         <generator class="native"/>
      </id>

      <property name = "firstName" column = "first_name" type = "string"/>
      <property name = "lastName" column = "last_name" type = "string"/>
      <property name = "salary" column = "salary" type = "int"/>

   </class>
</hibernate-mapping>
```

ManageEmployee.java

```java
import java.util.List;
import java.util.Date;
import java.util.Iterator;
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.SessionFactory;
import org.hibernate.Criteria;
import org.hibernate.criterion.Restrictions;
import org.hibernate.criterion.Projections;
import org.hibernate.cfg.Configuration;

public class ManageEmployee {
   private static SessionFactory factory;
   public static void main(String[] args) {

      try {
         factory = new Configuration().configure().buildSessionFactory();
      } catch (Throwable ex) {
         System.err.println("Failed to create sessionFactory object." + ex);
         throw new ExceptionInInitializerError(ex);
      }

      ManageEmployee ME = new ManageEmployee();

      /* Add few employee records in database */
      Integer empID1 = ME.addEmployee("Zara", "Ali", 2000);
      Integer empID2 = ME.addEmployee("Daisy", "Das", 5000);
      Integer empID3 = ME.addEmployee("John", "Paul", 5000);
      Integer empID4 = ME.addEmployee("Mohd", "Yasee", 3000);

      /* List down all the employees */
      ME.listEmployees();

      /* Print Total employee's count */
      ME.countEmployee();

      /* Print Total salary */
      ME.totalSalary();
   }

   /* Method to CREATE an employee in the database */
   public Integer addEmployee(String fname, String lname, int salary){
      Session session = factory.openSession();
      Transaction tx = null;
      Integer employeeID = null;

      try {
         tx = session.beginTransaction();
         Employee employee = new Employee(fname, lname, salary);
         employeeID = (Integer) session.save(employee);
         tx.commit();
      } catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace();
      } finally {
         session.close();
      }
      return employeeID;
   }

   /* Method to  READ all the employees having salary more than 2000 */
   public void listEmployees( ) {
      Session session = factory.openSession();
      Transaction tx = null;

      try {
         tx = session.beginTransaction();
         Criteria cr = session.createCriteria(Employee.class);
         // Add restriction.
         cr.add(Restrictions.gt("salary", 2000));
         List employees = cr.list();

         for (Iterator iterator = employees.iterator(); iterator.hasNext();){
            Employee employee = (Employee) iterator.next();
            System.out.print("First Name: " + employee.getFirstName());
            System.out.print("  Last Name: " + employee.getLastName());
            System.out.println("  Salary: " + employee.getSalary());
         }
         tx.commit();
      } catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace();
      } finally {
         session.close();
      }
   }

   /* Method to print total number of records */
   public void countEmployee(){
      Session session = factory.openSession();
      Transaction tx = null;

      try {
         tx = session.beginTransaction();
         Criteria cr = session.createCriteria(Employee.class);

         // To get total row count.
         cr.setProjection(Projections.rowCount());
         List rowCount = cr.list();

         System.out.println("Total Coint: " + rowCount.get(0) );
         tx.commit();
      } catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace();
      } finally {
         session.close();
      }
   }

   /* Method to print sum of salaries */
   public void totalSalary(){
      Session session = factory.openSession();
      Transaction tx = null;

      try {
         tx = session.beginTransaction();
         Criteria cr = session.createCriteria(Employee.class);

         // To get total salary.
         cr.setProjection(Projections.sum("salary"));
         List totalSalary = cr.list();

         System.out.println("Total Salary: " + totalSalary.get(0) );
         tx.commit();
      } catch (HibernateException e) {
         if (tx!=null) tx.rollback();
         e.printStackTrace();
      } finally {
         session.close();
      }
   }
}
```

Output

```java
cmd> java ManageEmployee
.......VARIOUS LOG MESSAGES WILL DISPLAY HERE........

First Name: Daisy  Last Name: Das  Salary: 5000
First Name: John  Last Name: Paul  Salary: 5000
First Name: Mohd  Last Name: Yasee  Salary: 3000
Total Coint: 4
Total Salary: 15000
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is a One-to-One association in Hibernate?

A **One-to-One** Association is similar to Many-to-One association with a difference that the column will be set as unique i.e. Two entities are said to be in a One-to-One relationship if one entity has only one occurrence in the other entity. For example, an address object can be associated with a single employee object. However, these relationships are rarely used in the relational table models and therefore, we won’t need this mapping too often.

In One-to-One association, the source entity has a field that references another target entity. The `@OneToOne` JPA annotation is used to map the source entity with the target entity.

Example: Hibernate One to One Mapping Annotation

**hibernate-annotation.cfg.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
		"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
		"http://hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.password">dbpassword</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost/TestDB</property>
        <property name="hibernate.connection.username">dbusername</property>
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>

        <property name="hibernate.current_session_context_class">thread</property>
        <property name="hibernate.show_sql">true</property>

        <mapping class="com.example.hibernate.model.Txn1"/>
        <mapping class="com.example.hibernate.model.Customer1"/>
    </session-factory>
</hibernate-configuration>
```

For hibernate one to one mapping annotation configuration, model classes are the most important part.

```java
package com.example.hibernate.model;

import java.util.Date;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.Table;
import org.hibernate.annotations.Cascade;

@Entity
@Table(name="TRANSACTION")
public class Txn1 {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="txn_id")
	private long id;

	@Column(name="txn_date")
	private Date date;

	@Column(name="txn_total")
	private double total;

	@OneToOne(mappedBy="txn")
	@Cascade(value=org.hibernate.annotations.CascadeType.SAVE_UPDATE)
	private Customer1 customer;

	@Override
	public String toString(){
		return id+", "+total+", "+customer.getName()+", "+customer.getEmail()+", "+customer.getAddress();
	}

        //Getter-Setter methods, omitted for clarity
}
```

```java

package com.example.hibernate.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.OneToOne;
import javax.persistence.PrimaryKeyJoinColumn;
import javax.persistence.Table;
import org.hibernate.annotations.GenericGenerator;
import org.hibernate.annotations.Parameter;

@Entity
@Table(name="CUSTOMER")
public class Customer1 {

	@Id
	@Column(name="txn_id", unique=true, nullable=false)
	@GeneratedValue(generator="gen")
	@GenericGenerator(name="gen", strategy="foreign", parameters={@Parameter(name="property", value="txn")})
	private long id;

	@Column(name="cust_name")
	private String name;

	@Column(name="cust_email")
	private String email;

	@Column(name="cust_address")
	private String address;

	@OneToOne
	@PrimaryKeyJoinColumn
	private Txn1 txn;

        //Getter-Setter methods
}
```

**Hibernate SessionFactory Utility class:**

```java

package com.example.hibernate.util;

import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;

public class HibernateAnnotationUtil {

	private static SessionFactory sessionFactory;

	private static SessionFactory buildSessionFactory() {
        try {
            // Create the SessionFactory from hibernate-annotation.cfg.xml
        	Configuration configuration = new Configuration();
        	configuration.configure("hibernate-annotation.cfg.xml");
        	System.out.println("Hibernate Annotation Configuration loaded");

        	ServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder().applySettings(configuration.getProperties()).build();
        	System.out.println("Hibernate Annotation serviceRegistry created");

        	SessionFactory sessionFactory = configuration.buildSessionFactory(serviceRegistry);

            return sessionFactory;
        }
        catch (Throwable ex) {
            System.err.println("Initial SessionFactory creation failed." + ex);
            ex.printStackTrace();
            throw new ExceptionInInitializerError(ex);
        }
    }

	public static SessionFactory getSessionFactory() {
		if(sessionFactory == null) sessionFactory = buildSessionFactory();
        return sessionFactory;
    }
}
```

**Hibernate One to One Mapping Annotation Example Test Program:**

```java

package com.example.hibernate.main;

import java.util.Date;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import com.example.hibernate.model.Customer1;
import com.example.hibernate.model.Txn1;
import com.example.hibernate.util.HibernateAnnotationUtil;

public class HibernateOneToOneAnnotationMain {

	public static void main(String[] args) {

		Txn1 txn = buildDemoTransaction();

		SessionFactory sessionFactory = null;
		Session session = null;
		Transaction tx = null;
		try{
		//Get Session
		sessionFactory = HibernateAnnotationUtil.getSessionFactory();
		session = sessionFactory.getCurrentSession();
		System.out.println("Session created using annotations configuration");
		//start transaction
		tx = session.beginTransaction();
		//Save the Model object
		session.save(txn);
		//Commit transaction
		tx.commit();
		System.out.println("Annotation Example. Transaction ID="+txn.getId());

		//Get Saved Trasaction Data
		printTransactionData(txn.getId(), sessionFactory);
		}catch(Exception e){
			System.out.println("Exception occured. "+e.getMessage());
			e.printStackTrace();
		}finally{
			if(sessionFactory != null && !sessionFactory.isClosed()){
				System.out.println("Closing SessionFactory");
				sessionFactory.close();
			}
		}
	}

	private static void printTransactionData(long id, SessionFactory sessionFactory) {
		Session session = null;
		Transaction tx = null;
		try{
			//Get Session
			sessionFactory = HibernateAnnotationUtil.getSessionFactory();
			session = sessionFactory.getCurrentSession();
			//start transaction
			tx = session.beginTransaction();
			//Save the Model object
			Txn1 txn = (Txn1) session.get(Txn1.class, id);
			//Commit transaction
			tx.commit();
			System.out.println("Annotation Example. Transaction Details=\n"+txn);

			}catch(Exception e){
				System.out.println("Exception occured. "+e.getMessage());
				e.printStackTrace();
			}
	}

	private static Txn1 buildDemoTransaction() {
		Txn1 txn = new Txn1();
		txn.setDate(new Date());
		txn.setTotal(100);

		Customer1 cust = new Customer1();
		cust.setAddress("San Jose, USA");
		cust.setEmail("Alex@yahoo.com");
		cust.setName("Alex Kr");

		txn.setCustomer(cust);

		cust.setTxn(txn);
		return txn;
	}
}
```

Output

```java
Hibernate Annotation Configuration loaded
Hibernate Annotation serviceRegistry created
Session created using annotations configuration
Hibernate: insert into TRANSACTION (txn_date, txn_total) values (?, ?)
Hibernate: insert into CUSTOMER (cust_address, cust_email, cust_name, txn_id) values (?, ?, ?, ?)
Annotation Example. Transaction ID=20
Hibernate: select txn1x0_.txn_id as txn_id1_1_0_, txn1x0_.txn_date as txn_date2_1_0_, txn1x0_.txn_total as txn_tota3_1_0_,
customer1x1_.txn_id as txn_id1_0_1_, customer1x1_.cust_address as cust_add2_0_1_, customer1x1_.cust_email as cust_ema3_0_1_,
customer1x1_.cust_name as cust_nam4_0_1_ from TRANSACTION txn1x0_ left outer join CUSTOMER customer1x1_ on
txn1x0_.txn_id=customer1x1_.txn_id where txn1x0_.txn_id=?
Annotation Example. Transaction Details=
20, 100.0, Alex Kr, Alex@yahoo.com, San Jose, USA
Closing SessionFactory
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is hibernate caching? Explain Hibernate first level cache?

Hibernate Cache can be very useful in gaining fast application performance if used correctly. The idea behind cache is to reduce the number of database queries, hence reducing the throughput time of the application.

Hibernate comes with different types of Cache:

**First Level Cache**: Hibernate first level cache is associated with the **Session object**. Hibernate uses this cache by default. Here, it processes one transaction after another one, means wont process one transaction many times. Mainly it reduces the number of SQL queries it needs to generate within a given transaction. That is instead of updating after every modification done in the transaction, it updates the transaction only at the end of the transaction.

**Second Level Cache**: Second-level cache always associates with the **Session Factory object**. While running the transactions, in between it loads the objects at the Session Factory level, so that those objects will be available to the entire application, not bound to single user. Since the objects are already loaded in the cache, whenever an object is returned by the query, at that time no need to go for a database transaction. In this way the second level cache works. Here we can use query level cache also.

Hibernate Second Level cache is disabled by default but we can enable it through configuration. Currently EHCache and Infinispan provides implementation for Hibernate Second level cache and we can use them. We will look into this in the next tutorial for hibernate caching.

**Query Cache**: Hibernate can also cache result set of a query. Hibernate Query Cache doesn’t cache the state of the actual entities in the cache; it caches only identifier values and results of value type. So it should always be used in conjunction with the second-level cache.

Example: Hibernate Caching – First Level Cache

```java

package com.example.hibernate.main;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

import com.example.hibernate.model.Employee;
import com.example.hibernate.util.HibernateUtil;

public class HibernateCacheExample {

	public static void main(String[] args) throws InterruptedException {

		SessionFactory sessionFactory = HibernateUtil.getSessionFactory();
		Session session = sessionFactory.getCurrentSession();
		Transaction tx = session.beginTransaction();

		//Get employee with id=1
		Employee emp = (Employee) session.load(Employee.class, new Long(1));
		printData(emp,1);

		//waiting for sometime to change the data in backend
		Thread.sleep(10000);

		//Fetch same data again, check logs that no query fired
		Employee emp1 = (Employee) session.load(Employee.class, new Long(1));
		printData(emp1,2);

		//Create new session
		Session newSession = sessionFactory.openSession();
		//Get employee with id=1, notice the logs for query
		Employee emp2 = (Employee) newSession.load(Employee.class, new Long(1));
		printData(emp2,3);

		//START: evict example to remove specific object from hibernate first level cache
		//Get employee with id=2, first time hence query in logs
		Employee emp3 = (Employee) session.load(Employee.class, new Long(2));
		printData(emp3,4);

		//evict the employee object with id=1
		session.evict(emp);
		System.out.println("Session Contains Employee with id=1?"+session.contains(emp));

		//since object is removed from first level cache, you will see query in logs
		Employee emp4 = (Employee) session.load(Employee.class, new Long(1));
		printData(emp4,5);

		//this object is still present, so you won't see query in logs
		Employee emp5 = (Employee) session.load(Employee.class, new Long(2));
		printData(emp5,6);
		//END: evict example

		//START: clear example to remove everything from first level cache
		session.clear();
		Employee emp6 = (Employee) session.load(Employee.class, new Long(1));
		printData(emp6,7);
		Employee emp7 = (Employee) session.load(Employee.class, new Long(2));
		printData(emp7,8);

		System.out.println("Session Contains Employee with id=2?"+session.contains(emp7));

		tx.commit();
		sessionFactory.close();
	}

	private static void printData(Employee emp, int count) {
		System.out.println(count+":: Name="+emp.getName()+", Zipcode="+emp.getAddress().getZipcode());
	}
}
```

Output

```java
Hibernate Configuration loaded
Hibernate serviceRegistry created
Hibernate: select employee0_.emp_id as emp_id1_1_0_, employee0_.emp_name as emp_name2_1_0_, employee0_.emp_salary as emp_sala3_1_0_, address1_.emp_id as emp_id1_0_1_, address1_.address_line1 as address_2_0_1_, address1_.city as city3_0_1_, address1_.zipcode as zipcode4_0_1_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
1:: Name=Alex, Zipcode=95129
2:: Name=Alex, Zipcode=95129
Hibernate: select employee0_.emp_id as emp_id1_1_0_, employee0_.emp_name as emp_name2_1_0_, employee0_.emp_salary as emp_sala3_1_0_, address1_.emp_id as emp_id1_0_1_, address1_.address_line1 as address_2_0_1_, address1_.city as city3_0_1_, address1_.zipcode as zipcode4_0_1_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
3:: Name=AlexK, Zipcode=95129
Hibernate: select employee0_.emp_id as emp_id1_1_0_, employee0_.emp_name as emp_name2_1_0_, employee0_.emp_salary as emp_sala3_1_0_, address1_.emp_id as emp_id1_0_1_, address1_.address_line1 as address_2_0_1_, address1_.city as city3_0_1_, address1_.zipcode as zipcode4_0_1_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
4:: Name=David, Zipcode=95051
Session Contains Employee with id=1?false
Hibernate: select employee0_.emp_id as emp_id1_1_0_, employee0_.emp_name as emp_name2_1_0_, employee0_.emp_salary as emp_sala3_1_0_, address1_.emp_id as emp_id1_0_1_, address1_.address_line1 as address_2_0_1_, address1_.city as city3_0_1_, address1_.zipcode as zipcode4_0_1_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
5:: Name=Alex, Zipcode=95129
6:: Name=David, Zipcode=95051
Hibernate: select employee0_.emp_id as emp_id1_1_0_, employee0_.emp_name as emp_name2_1_0_, employee0_.emp_salary as emp_sala3_1_0_, address1_.emp_id as emp_id1_0_1_, address1_.address_line1 as address_2_0_1_, address1_.city as city3_0_1_, address1_.zipcode as zipcode4_0_1_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
7:: Name=Alex, Zipcode=95129
Hibernate: select employee0_.emp_id as emp_id1_1_0_, employee0_.emp_name as emp_name2_1_0_, employee0_.emp_salary as emp_sala3_1_0_, address1_.emp_id as emp_id1_0_1_, address1_.address_line1 as address_2_0_1_, address1_.city as city3_0_1_, address1_.zipcode as zipcode4_0_1_ from EMPLOYEE employee0_ left outer join ADDRESS address1_ on employee0_.emp_id=address1_.emp_id where employee0_.emp_id=?
8:: Name=David, Zipcode=95051
Session Contains Employee with id=2?true
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is second level cache in Hibernate?

**Hibernate second level cache** uses a common cache for all the session object of a session factory. It is useful if you have multiple session objects from a session factory. **SessionFactory** holds the second level cache data. It is global for all the session objects and not enabled by default.

Different vendors have provided the implementation of Second Level Cache.

- EH Cache
- OS Cache
- Swarm Cache
- JBoss Cache

Each implementation provides different cache usage functionality. There are four ways to use second level cache.

- **read-only**: caching will work for read only operation.
- **nonstrict-read-write**: caching will work for read and write but one at a time.
- **read-write**: caching will work for read and write, can be used simultaneously.
- **transactional**: caching will work for transaction.

The cache-usage property can be applied to class or collection level in hbm.xml file.

```xml
<cache usage="read-only" />
```

Example: Hibernate Second Level Cache

Step 01: Create the persistent class using Maven

```java
package com.example;
import javax.persistence.*;
import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;
@Entity
@Table(name="emp1012")
@Cacheable
@Cache(usage=CacheConcurrencyStrategy.READ_ONLY)
public class Employee {
    @Id
   private int id;
   private String name;
   private float salary;

   public Employee() {}
   public Employee(String name, float salary) {
       super();
       this.name = name;
       this.salary = salary;
   }
   public int getId() {
       return id;
   }
   public void setId(int id) {
       this.id = id;
   }
   public String getName() {
       return name;
   }
   public void setName(String name) {
       this.name = name;
   }
   public float getSalary() {
       return salary;
   }
   public void setSalary(float salary) {
       this.salary = salary;
   }
}
```

Step 02: Add project information and configuration in pom.xml file.

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.2.16.Final</version>
</dependency>

<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc14</artifactId>
    <version>10.2.0.4.0</version>
</dependency>

<dependency>
    <groupId>net.sf.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>2.10.3</version>
</dependency>

<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-ehcache</artifactId>
    <version>5.2.16.Final</version>
</dependency>
```

Step 03: Create the Configuration file (hibernate.cfg.xml)

```xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
          "-//Hibernate/Hibernate Configuration DTD 5.2.0//EN"
          "http://hibernate.sourceforge.net/hibernate-configuration-5.2.0.dtd">

<hibernate-configuration>

    <session-factory>
        <property name="show_sql">true</property>
        <property name="hbm2ddl.auto">update</property>
        <property name="dialect">org.hibernate.dialect.Oracle9Dialect</property>
        <property name="connection.url">jdbc:oracle:thin:@localhost:1521:xe</property>
        <property name="connection.username">system</property>
        <property name="connection.password">jtp</property>
        <property name="connection.driver_class">oracle.jdbc.driver.OracleDriver</property>

         <property name="cache.use_second_level_cache">true</property>
         <property name="cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
         <mapping class="com.example.Employee"/>
    </session-factory>
</hibernate-configuration>
```

Step 04: Create the class that retrieves the persistent object

```java
package com.example;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.boot.Metadata;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;

public class FetchTest {

   public static void main(String[] args) {

       StandardServiceRegistry ssr = new StandardServiceRegistryBuilder().configure("hibernate.cfg.xml").build();
       Metadata meta = new MetadataSources(ssr).getMetadataBuilder().build();

       SessionFactory factory = meta.getSessionFactoryBuilder().build();

       Session session1 = factory.openSession();
       Employee emp1 = (Employee)session1.load(Employee.class,121);
       System.out.println(emp1.getId()+" "+emp1.getName()+" "+emp1.getSalary());
       session1.close();

       Session session2 = factory.openSession();
       Employee emp2 = (Employee)session2.load(Employee.class,121);
       System.out.println(emp2.getId()+" "+emp2.getName()+" "+emp2.getSalary());
       session2.close();

   }
}
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What are concurrency strategies?

The READ_WRITE strategy is an asynchronous cache concurrency mechanism and to prevent data integrity issues (e.g. stale cache entries), it uses a locking mechanism that provides unit-of-work isolation guarantees.

In hibernate, cache concurrency strategy can be set globally using the property hibernate.cache. default_cache_concurrency_strategy. The allowed values are,

- **read-only**: caching will work for read only operation. supported by ConcurrentHashMap, EHCache, Infinispan
- **nonstrict-read-write**: caching will work for read and write but one at a time. supported by ConcurrentHashMap, EHCache.
- **read-write**: caching will work for read and write, can be used simultaneously. supported by ConcurrentHashMap, EHCache.
- **transactional**: caching will work for transaction. supported by EHCache, Infinispan.

Example: Inserting data ( READ_WRITE strategy )

```java
@Override
public boolean afterInsert(
    Object key, Object value, Object version)
        throws CacheException {
    region().writeLock( key );
    try {
        final Lockable item =
            (Lockable) region().get( key );
        if ( item == null ) {
            region().put( key,
                new Item( value, version,
                    region().nextTimestamp()
                )
            );
            return true;
        }
        else {
            return false;
        }
    }
    finally {
        region().writeUnlock( key );
    }
}
```

For an entity to be cached upon insertion, it must use a SEQUENCE generator, the cache being populated by the EntityInsertAction:

```java
@Override
public void doAfterTransactionCompletion(boolean success,
    SessionImplementor session)
    throws HibernateException {

    final EntityPersister persister = getPersister();
    if ( success && isCachePutEnabled( persister,
        getSession() ) ) {
            final CacheKey ck = getSession()
               .generateCacheKey(
                    getId(),
                    persister.getIdentifierType(),
                    persister.getRootEntityName() );

            final boolean put = cacheAfterInsert(
                persister, ck );
        }
    }
    postCommitInsert( success );
}
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is Lazy loading in hibernate?

Hibernate defaults to a lazy fetching strategy for all entities and collections. Lazy loading in hibernate improves the performance. It loads the child objects on demand. To enable lazy loading explicitly you must use **fetch = FetchType.LAZY** on a association which you want to lazy load when you are using hibernate annotations.

Example:

```java
@OneToMany( mappedBy = "category", fetch = FetchType.LAZY )
private Set<ProductEntity> products;
```

## Q. Explain the persistent classes in Hibernate?

Persistence class are simple POJO classes in an application. It works as implementation of the business application for example Employee, department etc. It is not necessary that all instances of persistence class are defined persistence.

There are following main rules of persistent classes

- A persistence class should have a default constructor.
- A persistence class should have an id to uniquely identify the class objects.
- All attributes should be declared private.
- Public getter and setter methods should be defined to access the class attributes.

```java
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
import java.util.Date;

@Entity
@Table(name = "employee")
public class Employee {
    @Id
    @GeneratedValue
    @Column(name = "emp_id")
    private int id;

    @Column(name = "name")
    private String name;

    @Column(name = "salary")
    private int salary;

    @Column(name = "date_of_join")
    private Date dateOfJoin;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getSalary() {
        return salary;
    }

    public void setSalary(int salary) {
        this.salary = salary;
    }

    public Date getDateOfJoin() {
        return dateOfJoin;
    }

    public void setDateOfJoin(Date dateOfJoin) {
        this.dateOfJoin = dateOfJoin;
    }
}
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Explain some of the elements of hbm.xml?

Hibernate mapping file is used by hibernate framework to get the information about the mapping of a POJO class and a database table.

It mainly contains the following mapping information:

- Mapping information of a POJO class name to a database table name.
- Mapping information of POJO class properties to database table columns.

Elements of the Hibernate mapping file:

- **hibernate-mapping**: It is the root element.
- **Class**: It defines the mapping of a POJO class to a database table.
- **Id**: It defines the unique key attribute or primary key of the table.
- **generator**: It is the sub element of the id element. It is used to automatically generate the id.
- **property**: It is used to define the mapping of a POJO class property to database table column.

Syntax

```xml
<hibernate-mapping>

 <class name="POJO class name" table="table name in database">
  <id name="propertyName" column="columnName" type="propertyType" >
	<generator class="generatorClass"/>
  </id>
  <property name="propertyName1" column="colName1" type="propertyType " />
  <property name="propertyName2" column="colName2" type="propertyType " />
   . . . . .
 </class>

</hibernate-mapping>
```

## Q. What is Java Persistence API (JPA)?

Java Persistence API is a collection of classes and methods to persistently store the vast amounts of data into a database.
The Java Persistence API (JPA) is one possible approach to ORM. Via JPA the developer can map, store, update and retrieve data from relational databases to Java objects and vice versa. JPA can be used in Java-EE and Java-SE applications.

JPA metadata is typically defined via annotations in the Java class. Alternatively, the metadata can be defined via XML or a combination of both. A XML configuration overwrites the annotations.

**Relationship Mapping:**

JPA allows to define relationships between classes. Classes can have one to one, one to many, many to one, and many to many relationships with other classes. A relationship can be bidirectional or unidirectional, e.g. in a bidirectional relationship both classes store a reference to each other while in an unidirectional case only one class has a reference to the other class.

Relationship annotations:

- @OneToOne
- @OneToMany
- @ManyToOne
- @ManyToMany

![Java Exception](https://github.com/learning-zone/java-interview-questions/blob/master/assets/jpa_class_level_architecture.png)

**JPA - Architecture:**

| Units                | Description                                                                                                                      |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| EntityManagerFactory | This is a factory class of EntityManager. It creates and manages multiple EntityManager instances.                               |
| EntityManager        | It is an Interface, it manages the persistence operations on objects. It works like factory for Query instance.                  |
| Entity               | Entities are the persistence objects, stores as records in the database.                                                         |
| EntityTransaction    | It has one-to-one relationship with EntityManager. For each EntityManager, operations are maintained by EntityTransaction class. |
| Persistence          | This class contain static methods to obtain EntityManagerFactory instance.                                                       |
| Query                | This interface is implemented by each JPA vendor to obtain relational objects that meet the criteria.                            |

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Name some important interfaces of Hibernate framework?

- **Session Interface**: This is the primary interface used by hibernate applications
  The instances of this interface are lightweight and are inexpensive to create and destroy
  Hibernate sessions are not thread safe

- **Session Factory Interface**: This is a factory that delivers the session objects to hibernate application.

- **Configuration Interface**: This interface is used to configure and bootstrap hibernate.
  The instance of this interface is used by the application in order to specify the location of hbm documents

- **Transaction Interface**: This interface abstracts the code from any kind of transaction implementations such as JDBC transaction, JTA transaction

- **Query and Criteria Interface**: This interface allows the user to perform queries and also control the flow of the query execution

## Q. What is Hibernate SessionFactory and how to configure it?

SessionFactory can be created by providing Configuration object, which will contain all DB related property details pulled from either hibernate.cfg.xml file or hibernate.properties file. SessionFactory is a factory for Session objects.

We can create one SessionFactory implementation per database in any application. If your application is referring to multiple databases, then we need to create one SessionFactory per database.

The SessionFactory is a heavyweight object; it is usually created during application start up and kept for later use. The SessionFactory is a thread safe object and used by all the threads of an application.

```java
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;

import com.example.model.Employee;

public class HibernateUtil {

    private static SessionFactory sessionFactory = null;

    static {
        try {
            loadSessionFactory();
        } catch(Exception e){
            System.err.println("Exception while initializing hibernate util.. ");
            e.printStackTrace();
        }
    }

    public static void loadSessionFactory() {

        Configuration configuration = new Configuration();
        configuration.configure("/j2n-hibernate.cfg.xml");
        configuration.addAnnotatedClass(Employee.class);
        ServiceRegistry srvcReg = new StandardServiceRegistryBuilder().applySettings(configuration.getProperties()).build();
        sessionFactory = configuration.buildSessionFactory(srvcReg);
    }

    public static Session getSession() throws HibernateException {

        Session retSession=null;
            try {
                retSession = sessionFactory.openSession();
            } catch(Throwable t) {
                System.err.println("Exception while getting session.. ");
                t.printStackTrace();
            }
            if(retSession == null) {
                System.err.println("session is discovered null");
            }
            return retSession;
    }
}
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is the difference between opensession and getcurrentsession in hibernate?

Hibernate SessionFactory getCurrentSession() method returns the session bound to the context. Since this session object belongs to the hibernate context, we don't need to close it. Once the session factory is closed, this session object gets closed.

Hibernate SessionFactory openSession() method always opens a new session. We should close this session object once we are done with all the database operations.

| Parameter       | openSession                                                          | getCurrentSession                                                                                                                                         |
| --------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Session object  | It always create new Session object                                  | It creates a new Session if not exists , else use same session which is in current hibernate context                                                      |
| Flush and close | You need to explicitly flush and close session objects               | You do not need to flush and close session objects, it will be automatically taken care by Hibernate internally                                           |
| Performance     | In single threaded environment , It is slower than getCurrentSession | In single threaded environment , It is faster than getOpenSession                                                                                         |
| Configuration   | You do not need to configure any property to call this method        | You need to configure additional property “hibernate.current_session_context_class” to call getCurrentSession method, otherwise it will throw exceptions. |

Example: openSession()

```java
Transaction transaction = null;
Session session = HibernateUtil.getSessionFactory().openSession();
try {
    transaction = session.beginTransaction();
    // do Some work

    session.flush(); // extra work
    transaction.commit();
} catch(Exception ex) {
    if(transaction != null) {
        transaction.rollback();
    }
    ex.printStackTrace();
} finally {
    if(session != null) {
        session.close(); // extra work
    }
}
```

Example: getCurrentSession()

```java
Transaction transaction = null;
Session session = HibernateUtil.getSessionFactory().getCurrentSession();
try {
    transaction = session.beginTransaction();
    // do Some work

    // session.flush(); // no need
    transaction.commit();
} catch(Exception ex) {
    if(transaction != null) {
        transaction.rollback();
    }
    ex.printStackTrace();
} finally {
    if(session != null) {
        // session.close(); // no need
    }
}
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is difference between Hibernate Session get() and load() method?

Hibernate Session class provides two method to access object e.g. `session.get()` and `session.load()`.The difference between get() vs load() method is that get() involves database hit if object doesn't exists in **Session Cache** and returns a fully initialized object which may involve several database call while load method can return proxy in place and only initialize the object or hit the database if any method other than getId() is called on persistent or entity object. This lazy initialization can save couple of database round-trip which result in better performance.

| load()                                                                                             | get()                                                                          |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Only use the load() method if you are sure that the object exists.                                 | If you are not sure that the object exists, then use one of the get() methods. |
| load() method will throw an exception if the unique id is not found in the database.               | get() method will return null if the unique id is not found in the database.   |
| load() just returns a proxy by default and database won't be hit until the proxy is first invoked. | get() will hit the database immediately.                                       |

## Q. What are different states of an entity bean?

The Entity bean has three states:

**1. Transient**: Transient objects exist in heap memory. Hibernate does not manage transient objects or persist changes to transient objects. Whenever pojo class object created then it will be in the Transient state.

Transient state Object does not represent any row of the database, It means not associated with any Session object or no relation with the database its just an normal object.

**2. Persistent**: Persistent objects exist in the database, and Hibernate manages the persistence for persistent objects. If fields or properties change on a persistent object, Hibernate will keep the database representation up to date when the application marks the changes as to be committed.

Any instance returned by a **get()** or **load()** method is persistent state object.

**3. Detached**: Detached objects have a representation in the database, but changes to the object will not be reflected in the database, and vice-versa. A detached object can be created by closing the session that it was associated with, or by evicting it from the session with a call to the session's `evict()` method.

Example:

```java
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.cfg.AnnotationConfiguration;

import com.example.hibernate.Student;

public class ObjectStatesDemo {

    public static void main(String[] args) {

        // Transient object state
        Student student = new Student();
        student.setId(101);
        student.setName("Alex");
        student.setRoll("10");
        student.setDegree("B.E");
        student.setPhone("9999");

        // Transient object state
        Session session = new AnnotationConfiguration().configure()
                .buildSessionFactory().openSession();
        Transaction t = session.beginTransaction();

        // Persistent object state
        session.save(student);
        t.commit();

        // Detached object state
        session.close();
    }
}
```

Output

```java
Hibernate:
    insert
    into
        STUDENT
        (degree, name, phone, roll, id)
    values
        (?, ?, ?, ?, ?)
```

In The Database :

```sql
mysql> select * from student;
+-----+--------+--------+-------+------+
| id  | degree | name   | phone | roll |
+-----+--------+--------+-------+------+
| 101 | B.E    | Alex   | 9999  | 10   |
+-----+--------+--------+-------+------+
1 row in set (0.05 sec)
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is difference between merge() and update() methods in Hibernate?

Both update() and merge() methods in hibernate are used to convert the object which is in detached state into persistence state. But there are different situation where we should be used update() and where should be used merge() method in hibernate

```java
Employee emp1 = new Employee();
emp1.setEmpId(100);
emp1.setEmpName("Alex");
//create session
Session session1 = createNewHibernateSession();
session1.saveOrUpdate(emp1);
session1.close();
//emp1 object in detached state now

emp1.setEmpName("Alex Rajput");// Updated Name
//Create session again
Session session2 = createNewHibernateSession();
Employee emp2 =(Employee)session2.get(Employee.class, 100);
//emp2 object in persistent state with id 100

/**
Try to make on detached object with id 100 to persistent state by using update method of hibernate
It occurs the exception NonUniqueObjectException because emp2 object is having employee with same
empid as 100. In order to avoid this exception we are using merge like given below instead of
**/
session2.update(emp1);
session.update(emp1);

session2.merge(emp1); //it merge the object state with emp2
session2.update(emp1); //Now it will work with exception
```

In the hibernate session we can maintain only one employee object in persistent state with same primary key, while converting a detached object into persistent, if already that session has a persistent object with the same primary key then hibernate throws an Exception whenever update() method is called to reattach a detached object with a session. In this case we need to call **merge()** method instead of **update()** so that hibernate copies the state changes from detached object into persistent object and we can say a detached object is converted into a persistent object.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is difference between Hibernate save(), saveOrUpdate() and persist() methods?

All three methods (`save()`, `saveOrUpdate()`, and `persist()`) are used to make an entity persistent in Hibernate, but they differ in their behavior, return values, and when they can be used. Below is a detailed comparison:

| Method           | Return Value                                    | Usage                                                           | Behavior with Transient Entity                         | Behavior with Detached Entity                                                                                           | Exception Handling                                  |
| ---------------- | ----------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `save()`         | Returns the generated identifier (Serializable) | Used to save a new entity or reattach a detached entity         | Inserts the entity into the database and assigns an ID | If the entity has an ID, it may throw `NonUniqueObjectException` if another instance with the same ID is in the session | Throws exception if ID is already set and conflicts |
| `saveOrUpdate()` | Returns void                                    | Used when you don't know if the entity is transient or detached | Inserts the entity if transient                        | Updates the entity if detached                                                                                          | Handles both cases without exceptions               |
| `persist()`      | Returns void                                    | Used only for transient entities                                | Inserts the entity into the database                   | Throws `PersistenceException` if the entity is detached                                                                 | Throws exception for detached entities              |

**Key Differences:**

- **`save()`**: Always performs an insert operation and returns the ID. It can be used for both transient and detached entities, but for detached entities, it may lead to issues if the ID is already present in the session. It's part of the older Hibernate API.

- **`saveOrUpdate()`**: Intelligently decides whether to insert or update based on the entity's state. If the entity is transient (no ID or ID is null), it inserts; if detached (ID is set), it updates. This method is more flexible and safer for mixed scenarios.

- **`persist()`**: Only works with transient entities and does not return the ID. It's part of the JPA specification (via `EntityManager.persist()`). If called on a detached entity, it throws an exception. It's designed for pure insertion without side effects.

**Example Usage:**

```java
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

// Transient entity
Employee emp = new Employee("John", 50000);

// Using save()
Serializable id = session.save(emp); // Inserts and returns ID

// Using persist()
session.persist(emp); // Inserts, no return value

// Detached entity (after session close or evict)
session.evict(emp); // emp is now detached

// Using saveOrUpdate()
session.saveOrUpdate(emp); // Updates the entity

tx.commit();
session.close();
```

**When to Use:**

- Use `save()` when you need the generated ID immediately and are sure about the entity's state.
- Use `saveOrUpdate()` for generic save operations where the entity might be new or modified.
- Use `persist()` in JPA-compliant code for inserting new entities, ensuring no detached entities are passed.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What will happen if we don’t have no-args constructor in Entity bean?

In Hibernate (and JPA), an Entity bean (or entity class) must have a no-argument constructor (also known as a default constructor). This is a mandatory requirement for all JPA-compliant implementations, including Hibernate. If an entity class does not have a no-args constructor, Hibernate will throw an exception during runtime when attempting to instantiate the entity.

**Why is a No-Args Constructor Required?**

- **Reflection and Instantiation**: Hibernate uses Java Reflection API to create instances of entity classes. The `Class.newInstance()` method (or similar mechanisms in newer Java versions) requires a no-args constructor to instantiate objects without parameters.
- **Proxy Generation**: For lazy loading and other Hibernate features, Hibernate may generate proxy subclasses of the entity. These proxies also need to call the no-args constructor of the superclass (the entity).
- **JPA Specification**: The JPA specification explicitly states that all entity classes must have a public or protected no-argument constructor.

**What Happens If We Don’t Have a No-Args Constructor?**

If an entity class lacks a no-args constructor, Hibernate will fail to instantiate the entity, leading to runtime exceptions such as:

- `InstantiationException`: Thrown when Hibernate tries to create an instance using reflection but cannot find a suitable constructor.
- `NoSuchMethodException`: Indicates that the no-args constructor is missing.
- `HibernateException` or `PersistenceException`: Wrapping the underlying reflection errors.

These exceptions typically occur during operations like `session.get()`, `session.load()`, or when executing queries that return entity instances.

**Example Scenario:**

Consider an entity class without a no-args constructor:

```java
@Entity
@Table(name = "employee")
public class Employee {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    // Parameterized constructor only
    public Employee(String name) {
        this.name = name;
    }

    // Getters and setters omitted for brevity
}
```

When Hibernate tries to load this entity:

```java
Session session = sessionFactory.openSession();
Employee emp = session.get(Employee.class, 1L); // This will fail
```

**Output/Error:**

```
org.hibernate.InstantiationException: Unable to instantiate default constructor for class com.example.Employee
    at org.hibernate.tuple.PojoInstantiator.instantiate(PojoInstantiator.java:87)
    ...
Caused by: java.lang.NoSuchMethodException: com.example.Employee.<init>()
    at java.lang.Class.getConstructor0(Class.java:3082)
    ...
```

**How to Fix It:**

Always provide a no-args constructor in your entity classes. It can be public or protected, but not private (as Hibernate needs to access it via reflection).

```java
@Entity
@Table(name = "employee")
public class Employee {
    @Id
    @GeneratedValue
    private Long id;

    private String name;

    // No-args constructor (required)
    public Employee() {
    }

    // Parameterized constructor (optional)
    public Employee(String name) {
        this.name = name;
    }

    // Getters and setters
}
```

**Best Practices:**

- Always include a no-args constructor in entity classes, even if you have parameterized constructors.
- If using Lombok, you can use `@NoArgsConstructor` annotation to generate it automatically.
- Avoid making the no-args constructor private, as it will prevent Hibernate from accessing it.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is difference between sorted collection and ordered collection, which one is better?

In Hibernate, collections can be sorted or ordered based on specific criteria. The key difference lies in **when and where** the sorting occurs: in the database (ordered) or in memory (sorted). Understanding these concepts helps in optimizing performance and choosing the right approach for your use case.

**Sorted Collection:**

- Sorting is performed in memory by Hibernate after the data is loaded from the database.
- Uses a `Comparator` or the natural ordering of the elements (via `Comparable` interface).
- The SQL query does not include an ORDER BY clause; sorting happens post-query.
- Suitable for small collections where in-memory sorting is acceptable.
- Can be customized with complex sorting logic that might not be easily expressible in SQL.

**Ordered Collection:**

- Sorting is performed by the database using an ORDER BY clause in the SQL query.
- The results are already sorted when retrieved from the database.
- More efficient for large datasets, as the database handles the sorting.
- Relies on database capabilities for ordering, which is typically faster for large volumes of data.

**Key Differences:**

| Aspect                   | Sorted Collection                              | Ordered Collection                          |
| ------------------------ | ---------------------------------------------- | ------------------------------------------- |
| **Where Sorting Occurs** | In memory (after loading from DB)              | In database (via SQL ORDER BY)              |
| **Performance**          | Slower for large collections (memory overhead) | Faster for large collections (DB optimized) |
| **SQL Query**            | No ORDER BY clause                             | Includes ORDER BY clause                    |
| **Customization**        | Highly flexible (custom Comparator)            | Limited to SQL-supported ordering           |
| **Use Case**             | Small collections, complex sorting logic       | Large collections, simple ordering          |
| **Memory Usage**         | Higher (loads all data before sorting)         | Lower (sorted at DB level)                  |
| **Database Load**        | Lower (sorting in app)                         | Higher (sorting in DB)                      |

**Which One is Better?**

Neither is universally "better"—it depends on your specific requirements:

- **Use Ordered Collection** if:

  - You have large collections and performance is critical.
  - The sorting criteria are simple and can be expressed in SQL (e.g., by a single column).
  - You want to minimize memory usage in your application.

- **Use Sorted Collection** if:
  - Collections are small, and in-memory sorting won't impact performance.
  - You need complex, custom sorting logic (e.g., multi-level sorting or business-specific rules) that SQL can't easily handle.
  - You prefer keeping sorting logic in the application layer for maintainability.

In general, for most enterprise applications with potentially large datasets, **ordered collections are preferred** because they leverage database efficiency. However, always profile your application to confirm.

**Example: Sorted Collection**

```java
@Entity
public class Department {
    @Id
    private Long id;

    @OneToMany(mappedBy = "department")
    @Sort(type = SortType.COMPARATOR, comparator = EmployeeComparator.class)
    private SortedSet<Employee> employees = new TreeSet<>();
}

public class EmployeeComparator implements Comparator<Employee> {
    @Override
    public int compare(Employee e1, Employee e2) {
        return e1.getName().compareTo(e2.getName());
    }
}
```

Here, employees are sorted in memory using the custom comparator.

**Example: Ordered Collection**

```java
@Entity
public class Department {
    @Id
    private Long id;

    @OneToMany(mappedBy = "department")
    @OrderBy("name ASC")
    private List<Employee> employees = new ArrayList<>();
}
```

The SQL query will include `ORDER BY name ASC`, sorting at the database level.

**Best Practices:**

- For large datasets, prefer ordered collections to avoid loading unnecessary data into memory.
- Use sorted collections only when the collection size is small or when custom logic is required.
- Consider indexing the ordered columns in the database for better performance.
- Test both approaches in your environment to measure performance differences.
- If using JPA annotations, `@OrderBy` is for ordered collections, while `@Sort` is Hibernate-specific for sorted collections.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What are the collection types in Hibernate?

In Hibernate, collections are used to represent associations between entities, such as one-to-many or many-to-many relationships. Hibernate supports several collection types that map to Java collections and are persisted to the database. The main collection types are:

1. **Bag**:

   - Mapped to `java.util.List` or `java.util.Collection`.
   - Allows duplicates and does not maintain order.
   - Suitable for unordered collections where duplicates are allowed.
   - Example: `<bag name="items" table="item_table">`

2. **Set**:

   - Mapped to `java.util.Set`.
   - Does not allow duplicates and does not maintain order.
   - Uses a join table or foreign key for many-to-many or one-to-many.
   - Example: `<set name="tags" table="tag_table">`

3. **List**:

   - Mapped to `java.util.List`.
   - Maintains order and allows duplicates.
   - Requires an index column to preserve order.
   - Example: `<list name="tasks" table="task_table"> <key column="user_id"/> <index column="position"/> <element column="task_name"/> </list>`

4. **Map**:

   - Mapped to `java.util.Map`.
   - Stores key-value pairs.
   - Requires a key column and a value column.
   - Example: `<map name="properties" table="property_table"> <key column="entity_id"/> <map-key column="key_column"/> <element column="value_column"/> </map>`

5. **Array**:
   - Mapped to arrays (e.g., `String[]`).
   - Rarely used, as it's less flexible than other collections.
   - Example: `<array name="names" table="name_table">`

These collection types are defined in Hibernate mapping files (hbm.xml) or using annotations like `@OneToMany`, `@ManyToMany`, etc. Choosing the right collection type depends on the requirements for order, duplicates, and performance.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How to implement Joins in Hibernate?

In Hibernate, joins can be implemented using HQL (Hibernate Query Language), Criteria API, or through associations defined in entity mappings. Joins allow you to retrieve data from multiple related tables in a single query, improving performance by reducing the number of database hits.

### 1. Using HQL (Hibernate Query Language)

HQL is similar to SQL but operates on entities and their properties. You can perform INNER JOIN, LEFT JOIN, RIGHT JOIN, etc.

**Example: INNER JOIN**

```java
String hql = "FROM Employee e INNER JOIN e.department d WHERE d.name = :deptName";
Query query = session.createQuery(hql);
query.setParameter("deptName", "IT");
List<Object[]> results = query.list(); // Returns array of Employee and Department
```

**Example: LEFT JOIN**

```java
String hql = "FROM Employee e LEFT JOIN e.department d";
Query query = session.createQuery(hql);
List<Object[]> results = query.list();
```

**Fetching with SELECT**

```java
String hql = "SELECT e.name, d.name FROM Employee e JOIN e.department d";
Query query = session.createQuery(hql);
List<Object[]> results = query.list();
```

### 2. Using Criteria API

The Criteria API provides a programmatic way to build queries with joins.

**Example:**

```java
Criteria criteria = session.createCriteria(Employee.class, "e");
criteria.createAlias("e.department", "d"); // Creates join
criteria.add(Restrictions.eq("d.name", "IT"));
List<Employee> employees = criteria.list();
```

For projections:

```java
Criteria criteria = session.createCriteria(Employee.class, "e");
criteria.createAlias("e.department", "d");
criteria.setProjection(Projections.projectionList()
    .add(Projections.property("e.name"))
    .add(Projections.property("d.name")));
List<Object[]> results = criteria.list();
```

### 3. Using Associations (Automatic Joins)

When you define relationships in your entities using annotations like `@OneToMany`, `@ManyToOne`, Hibernate automatically handles joins when you access related entities.

**Example Entity Mapping:**

```java
@Entity
public class Employee {
    @Id
    private Long id;

    @ManyToOne
    @JoinColumn(name = "dept_id")
    private Department department;

    // getters and setters
}

@Entity
public class Department {
    @Id
    private Long id;

    @OneToMany(mappedBy = "department")
    private List<Employee> employees;

    // getters and setters
}
```

**Fetching with Join:**

```java
Employee emp = (Employee) session.get(Employee.class, 1L);
Department dept = emp.getDepartment(); // Hibernate performs join automatically
```

### Types of Joins

- **INNER JOIN**: Returns only matching rows from both tables.
- **LEFT JOIN**: Returns all rows from the left table and matching rows from the right table.
- **RIGHT JOIN**: Returns all rows from the right table and matching rows from the left table.
- **FULL JOIN**: Returns all rows from both tables.

### Best Practices

- Use joins to minimize database queries and avoid N+1 problems.
- Prefer HQL or Criteria for complex queries.
- Use fetch joins (e.g., `JOIN FETCH` in HQL) to eagerly load related entities in a single query.
- Monitor generated SQL to ensure efficient execution.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Why we should not make Entity Class final?

**Answer:** Making an entity class final in Hibernate is not recommended because Hibernate relies on proxy generation for features like lazy loading. When an entity is marked as final, Hibernate cannot create a subclass proxy, which is essential for intercepting method calls to implement lazy initialization. This can lead to issues where associated entities are loaded eagerly instead of lazily, potentially causing performance problems. Additionally, Hibernate uses proxies for other purposes, such as change detection and caching. By keeping the entity class non-final, Hibernate can dynamically generate proxies at runtime, ensuring optimal performance and functionality.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is the benefit of native sql query support in hibernate?

**Answer:** Hibernate provides support for native SQL queries, which allows developers to execute raw SQL statements directly against the database. This is beneficial in several scenarios:

1. **Complex Queries:** When HQL or JPQL cannot express complex queries involving database-specific features like stored procedures, window functions, or advanced joins, native SQL can be used.

2. **Performance Optimization:** Native SQL gives direct control over the SQL generated, allowing for fine-tuning of queries for better performance, especially when dealing with large datasets or specific database optimizations.

3. **Database-Specific Features:** It enables the use of database-specific extensions and functions that are not supported by HQL.

4. **Migration from JDBC:** For applications migrating from JDBC to Hibernate, native SQL provides a smoother transition by allowing existing SQL queries to be reused.

5. **Stored Procedures:** Native SQL support makes it easier to call stored procedures and handle their results within Hibernate.

However, using native SQL should be done judiciously as it can reduce portability across different databases and may bypass some of Hibernate's ORM features like automatic dirty checking and caching.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is Named SQL Query? What are the benefits of Named SQL Query?

**Answer:** Named SQL queries in Hibernate are predefined SQL queries that are defined in Hibernate mapping files (such as hbm.xml) or using annotations, and are assigned a unique name. This allows developers to reference and execute these queries by name in the application code, rather than writing raw SQL each time. Named queries can include parameters, making them flexible and reusable.

**Benefits of Named SQL Query:**

- **Reusability:** Define a query once and reuse it across multiple parts of the application, reducing code duplication.
- **Performance:** Named queries are precompiled by Hibernate, which can improve execution speed compared to dynamically created queries.
- **Maintainability:** Centralized query management makes it easier to update and maintain SQL logic in one place.
- **Security:** Supports parameterized queries, helping to prevent SQL injection attacks.
- **Readability:** Improves code readability by separating SQL from Java code.

**Example:**

Define a named query in `hibernate.cfg.xml` or an entity mapping file:

```xml
<hibernate-mapping>
    <sql-query name="findEmployeesByDept">
        <return alias="emp" class="com.example.Employee"/>
        SELECT emp_id, emp_name, emp_salary FROM EMPLOYEE WHERE dept_id = ?
    </sql-query>
</hibernate-mapping>
```

Or using annotations in an entity class:

```java
@NamedNativeQuery(
    name = "findEmployeesByDept",
    query = "SELECT * FROM EMPLOYEE WHERE dept_id = ?",
    resultClass = Employee.class
)
```

Execute the named query in Java code:

```java
Session session = sessionFactory.openSession();
Query query = session.getNamedQuery("findEmployeesByDept");
query.setParameter(1, deptId);
List<Employee> employees = query.list();
```

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is cascading and what are different types of cascading?

**Answer:** Cascading in Hibernate refers to the automatic propagation of operations (like save, update, delete) from a parent entity to its associated child entities. This ensures that when you perform an operation on the parent, the same operation is applied to the related entities, maintaining data integrity and reducing boilerplate code.

### Types of Cascading

Hibernate supports several cascade types, defined in the `javax.persistence.CascadeType` enum or Hibernate's `org.hibernate.annotations.CascadeType`. You can apply them using annotations like `@OneToMany(cascade = CascadeType.ALL)` or in XML mappings.

1. **ALL**: Propagates all operations (PERSIST, MERGE, REMOVE, REFRESH, DETACH).
2. **PERSIST**: Propagates save/persist operations.
3. **MERGE**: Propagates merge operations.
4. **REMOVE**: Propagates delete/remove operations.
5. **REFRESH**: Propagates refresh operations (reload from database).
6. **DETACH**: Propagates detach operations (remove from session).

### Examples

Consider entities `Parent` and `Child` with a one-to-many relationship.

**Entity Definitions:**

```java
@Entity
public class Parent {
    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> children = new ArrayList<>();
}

@Entity
public class Child {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    private Parent parent;
}
```

**Example 1: CascadeType.PERSIST**

```java
Parent parent = new Parent();
Child child = new Child();
child.setParent(parent);
parent.getChildren().add(child);

session.persist(parent); // Saves both parent and child
```

**Example 2: CascadeType.REMOVE**

```java
Parent parent = session.get(Parent.class, 1L);
session.remove(parent); // Deletes parent and all its children
```

**Example 3: CascadeType.MERGE**

```java
Parent detachedParent = new Parent();
detachedParent.setId(1L);
Child newChild = new Child();
newChild.setParent(detachedParent);
detachedParent.getChildren().add(newChild);

Parent mergedParent = (Parent) session.merge(detachedParent); // Merges parent and saves new child
```

**Example 4: CascadeType.REFRESH**

```java
Parent parent = session.get(Parent.class, 1L);
// Assume data changes in DB
session.refresh(parent); // Refreshes parent and all children from DB
```

**Example 5: CascadeType.DETACH**

```java
Parent parent = session.get(Parent.class, 1L);
session.detach(parent); // Detaches parent and all children from session
```

### Best Practices

- Use cascading judiciously to avoid unintended operations.
- For complex relationships, consider using specific cascade types instead of `ALL`.
- Be aware of potential performance implications, like cascading deletes on large datasets.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How to integrate log4j logging in hibernate application?

**Answer:** To integrate Log4j logging in a Hibernate application, follow these steps:

### 1. Add Log4j Dependencies

Include Log4j dependencies in your project's build file (e.g., pom.xml for Maven).

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.1</version>
</dependency>
```

### 2. Configure Log4j

Create a `log4j2.xml` configuration file in your classpath (e.g., src/main/resources).

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
        <File name="FileAppender" fileName="logs/hibernate.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </File>
    </Appenders>
    <Loggers>
        <Logger name="org.hibernate" level="DEBUG" additivity="false">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="FileAppender"/>
        </Logger>
        <Root level="INFO">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

### 3. Configure Hibernate to Use Log4j

In your Hibernate configuration (hibernate.cfg.xml), ensure that Hibernate uses SLF4J, which bridges to Log4j.

```xml
<property name="hibernate.show_sql">true</property>
<property name="hibernate.format_sql">true</property>
```

### 4. Add SLF4J Bridge for Log4j

If using Log4j 2, add the SLF4J bridge.

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
    <version>2.17.1</version>
</dependency>
```

### 5. Example Usage in Code

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HibernateExample {
    private static final Logger logger = LoggerFactory.getLogger(HibernateExample.class);

    public static void main(String[] args) {
        // Hibernate operations
        logger.info("Starting Hibernate application");
        // Your Hibernate code here
        logger.debug("Hibernate operation completed");
    }
}
```

This setup will log Hibernate operations to both console and file as configured.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Which design patterns are used in Hibernate framework?

**Answer:** Hibernate framework uses several design patterns to provide a robust and flexible ORM solution. The key design patterns used in Hibernate include:

1. **Factory Pattern:** Used in the creation of `SessionFactory` and `Session` objects. The `SessionFactory` is a factory for `Session` instances, abstracting the creation process.

2. **Proxy Pattern:** Hibernate uses proxies to implement lazy loading. It creates proxy objects for entities to defer database loading until the object is actually accessed.

3. **Template Method Pattern:** The `HibernateTemplate` class in Spring uses this pattern to provide a template for common Hibernate operations, handling session management and exception translation.

4. **Data Access Object (DAO) Pattern:** Encourages separation of data access logic from business logic by encapsulating database operations in DAO classes.

5. **Singleton Pattern:** The `SessionFactory` is typically implemented as a singleton to ensure a single instance per application.

6. **Decorator Pattern:** Used in wrapping JDBC connections and statements to add additional functionality like logging or transaction management.

7. **Observer Pattern:** Hibernate uses event listeners and callbacks to observe and react to entity lifecycle events.

These patterns help Hibernate manage complexity, improve maintainability, and provide extensibility.

## Q. What is cascading and what are different types of cascading? Give examples.

**Answer:** Cascading in Hibernate refers to the automatic propagation of operations (like save, update, delete) from a parent entity to its associated child entities. This ensures that when you perform an operation on the parent, the same operation is applied to the related entities, maintaining data integrity and reducing boilerplate code.

### Types of Cascading

Hibernate supports several cascade types, defined in the `javax.persistence.CascadeType` enum or Hibernate's `org.hibernate.annotations.CascadeType`. You can apply them using annotations like `@OneToMany(cascade = CascadeType.ALL)` or in XML mappings.

1. **ALL**: Propagates all operations (PERSIST, MERGE, REMOVE, REFRESH, DETACH).
2. **PERSIST**: Propagates save/persist operations.
3. **MERGE**: Propagates merge operations.
4. **REMOVE**: Propagates delete/remove operations.
5. **REFRESH**: Propagates refresh operations (reload from database).
6. **DETACH**: Propagates detach operations (remove from session).

### Examples

Consider entities `Parent` and `Child` with a one-to-many relationship.

**Entity Definitions:**

```java
@Entity
public class Parent {
    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> children = new ArrayList<>();
}

@Entity
public class Child {
    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    private Parent parent;
}
```

**Example 1: CascadeType.PERSIST**

```java
Parent parent = new Parent();
Child child = new Child();
child.setParent(parent);
parent.getChildren().add(child);

session.persist(parent); // Saves both parent and child
```

**Example 2: CascadeType.REMOVE**

```java
Parent parent = session.get(Parent.class, 1L);
session.remove(parent); // Deletes parent and all its children
```

**Example 3: CascadeType.MERGE**

```java
Parent detachedParent = new Parent();
detachedParent.setId(1L);
Child newChild = new Child();
newChild.setParent(detachedParent);
detachedParent.getChildren().add(newChild);

Parent mergedParent = (Parent) session.merge(detachedParent); // Merges parent and saves new child
```

**Example 4: CascadeType.REFRESH**

```java
Parent parent = session.get(Parent.class, 1L);
// Assume data changes in DB
session.refresh(parent); // Refreshes parent and all children from DB
```

**Example 5: CascadeType.DETACH**

```java
Parent parent = session.get(Parent.class, 1L);
session.detach(parent); // Detaches parent and all children from session
```

### Best Practices

- Use cascading judiciously to avoid unintended operations.
- For complex relationships, consider using specific cascade types instead of `ALL`.
- Be aware of potential performance implications, like cascading deletes on large datasets.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How to log hibernate generated sql queries in log files?

**Answer:** To log Hibernate-generated SQL queries in log files, you need to configure Hibernate properties and logging frameworks appropriately. Here's a step-by-step guide:

### 1. Enable SQL Logging in Hibernate Configuration

Set the following properties in your `hibernate.cfg.xml` or `application.properties` (for Spring Boot):

**hibernate.cfg.xml:**

```xml
<property name="hibernate.show_sql">true</property>
<property name="hibernate.format_sql">true</property>
```

**application.properties (Spring Boot):**

```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

- `hibernate.show_sql=true`: Prints SQL statements to the console.
- `hibernate.format_sql=true`: Formats the SQL for better readability.

### 2. Configure Logging Framework

To log to files instead of console, configure your logging framework (e.g., Log4j, Logback, or SLF4J).

**Using Log4j (log4j.properties):**

```properties
# Set logging level for Hibernate SQL
log4j.logger.org.hibernate.SQL=DEBUG
log4j.logger.org.hibernate.type.descriptor.sql.BasicBinder=TRACE

# Configure file appender
log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=hibernate.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

**Using Logback (logback.xml):**

```xml
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>hibernate.log</file>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="org.hibernate.SQL" level="DEBUG" additivity="false">
        <appender-ref ref="FILE"/>
    </logger>
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" additivity="false">
        <appender-ref ref="FILE"/>
    </logger>

    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### 3. Log Query Parameters

To log actual parameter values (e.g., `?` in prepared statements), set the logging level for `org.hibernate.type.descriptor.sql.BasicBinder` to `TRACE`.

### Example Output

With the above configuration, your log file might contain:

```
2023-10-01 12:34:56 DEBUG org.hibernate.SQL - select employee0_.id as id1_0_, employee0_.name as name2_0_ from Employee employee0_ where employee0_.id=?
2023-10-01 12:34:56 TRACE org.hibernate.type.descriptor.sql.BasicBinder - binding parameter [1] as [BIGINT] - [123]
```

### Note

- Logging SQL can impact performance in production; use it only for debugging.
- For Spring Boot, you can also use `logging.level.org.hibernate.SQL=DEBUG` in `application.properties`.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What are the technologies that are supported by Hibernate?

**Answer:** Hibernate is a powerful ORM (Object-Relational Mapping) framework for Java, and it supports integration with several key technologies and frameworks to enhance enterprise application development.

**✅ Supported Technologies:**

1. Java/J2EE
   Hibernate is written in Java and works seamlessly with Java SE and Java EE applications.

2. JDBC (Java Database Connectivity)
   Hibernate internally uses JDBC to interact with relational databases. It abstracts the boilerplate JDBC code and provides a cleaner API.

3. JPA (Java Persistence API)
   Hibernate implements the JPA specification, allowing developers to write portable and standard persistence code.

4. Spring Framework
   Hibernate integrates well with Spring for dependency injection, transaction management, and simplified configuration.

5. Servlets and JSP
   Hibernate can be used in web applications built using Servlets and JSP for data persistence.

6. Struts Framework
   Hibernate can be integrated with Struts to handle persistence in MVC-based web applications.

7. EJB (Enterprise JavaBeans)
   Hibernate can also work within EJB containers for enterprise applications.

8. Various Databases
   Hibernate supports all major relational databases like MySQL, PostgreSQL, Oracle, SQL Server, H2, and more.

9. Caching Frameworks
   Hibernate supports second-level caching using frameworks like Ehcache, Infinispan, or OSCache.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is your understanding of Hibernate proxy?

**Answer:** In Hibernate, a proxy is a placeholder object created by Hibernate instead of fetching the actual entity data immediately. It is mainly used to support lazy loading, which means that the associated entity data is only fetched when it’s accessed for the first time.

**🔍 How It Works:**

- When you retrieve an entity using session.load() instead of session.get(), Hibernate returns a proxy object.

- This proxy is a subclass of your entity and contains just the entity’s identifier (primary key).

- When any non-identifier method is called on the proxy, Hibernate fetches the actual data from the database.

```java
Student student = session.load(Student.class, 1); // Returns proxy, not actual object

// At this point, no SQL query is executed yet
System.out.println(student.getId()); // Uses proxy (no DB hit)

// When accessing other properties, Hibernate fetches data from DB
System.out.println(student.getName()); // SQL query executed here
```

**Benefits of Hibernate Proxy:**

- Improves performance by loading data only when needed.

- Reduces memory usage for large object graphs.

- Helps in managing associations without unnecessary joins.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Can you explain Hibernate callback interfaces?

**Answer:** Hibernate provides callback interfaces that allow developers to intercept and respond to specific lifecycle events of entity objects. These callbacks are useful for performing custom logic before or after an entity is persisted, updated, deleted, or loaded.

**🧩 Common Callback Interfaces:**

| Interface                         | Description                                                                   |
| --------------------------------- | ----------------------------------------------------------------------------- |
| `org.hibernate.Callback`          | Marker interface for callback-enabled classes.                                |
| `org.hibernate.classic.Lifecycle` | Provides methods like `onSave()`, `onUpdate()`, `onDelete()`, and `onLoad()`. |
| `org.hibernate.Interceptor`       | Used to intercept and customize persistence operations globally.              |
| `org.hibernate.event.*`           | Event listeners for specific entity lifecycle events.                         |

**Example: Using Lifecycle Interface**

```java
import org.hibernate.classic.Lifecycle;
import org.hibernate.Session;
import org.hibernate.HibernateException;
import java.io.Serializable;

public class Student implements Lifecycle {
    private int id;
    private String name;

    // Getters and Setters

    @Override
    public boolean onSave(Session s) throws HibernateException {
        System.out.println("Before saving student: " + name);
        return false; // return true to veto the save
    }

    @Override
    public boolean onUpdate(Session s) throws HibernateException {
        System.out.println("Before updating student: " + name);
        return false;
    }

    @Override
    public boolean onDelete(Session s) throws HibernateException {
        System.out.println("Before deleting student: " + name);
        return false;
    }

    @Override
    public void onLoad(Session s, Serializable id) throws HibernateException {
        System.out.println("Student loaded with ID: " + id);
    }
}
```

**Explanation:**

- onSave() is called before the entity is saved.

- onUpdate() is called before the entity is updated.

- onDelete() is called before the entity is deleted.

- onLoad() is called after the entity is loaded from the database.

**Example: Using Interceptor:**

```java
import org.hibernate.EmptyInterceptor;
import org.hibernate.type.Type;
import java.io.Serializable;

public class MyInterceptor extends EmptyInterceptor {
    @Override
    public boolean onSave(Object entity, Serializable id, Object[] state, String[] propertyNames, Type[] types) {
        System.out.println("Entity is being saved: " + entity.getClass().getName());
        return super.onSave(entity, id, state, propertyNames, types);
    }
}
```

**Then register the interceptor:**

```java
Session session = sessionFactory
    .withOptions()
    .interceptor(new MyInterceptor())
    .openSession();
```

**Why Use Callback Interfaces?**

- Automate logging or auditing of entity lifecycle events.

- Perform validation or transformation before persistence.

- Trigger external processes or notifications on entity state changes.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How to create database applications in Java with the use of Hibernate?

**Answer:** Creating database applications with Hibernate involves using its powerful ORM (Object-Relational Mapping) capabilities to map Java objects to relational database tables. Hibernate eliminates the need for writing complex JDBC code and provides an abstraction layer to interact with the database in an object-oriented way.

**🧩 Steps to Create a Database Application using Hibernate:**

1. Add Hibernate Dependencies

- If using Maven, include the following dependency:

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.15.Final</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
```

2. Create the Hibernate Configuration File (hibernate.cfg.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/hibernatedb</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">password</property>

        <!-- Hibernate settings -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</property>
        <property name="hibernate.hbm2ddl.auto">update</property>
        <property name="show_sql">true</property>

        <!-- Mapping class -->
        <mapping class="com.example.Student"/>
    </session-factory>
</hibernate-configuration>
```

3. Create an Entity Class

```java
import javax.persistence.*;

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    @Column(name = "name")
    private String name;

    @Column(name = "email")
    private String email;

    // Getters and Setters
}
```

4. Create the Main Class

```java
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class MainApp {
    public static void main(String[] args) {
        SessionFactory factory = new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();
        Session session = factory.openSession();

        Student student = new Student();
        student.setName("John Doe");
        student.setEmail("john@example.com");

        session.beginTransaction();
        session.save(student);
        session.getTransaction().commit();

        session.close();
        factory.close();
    }
}
```

##### Result: A record is automatically inserted into the students table without writing a single SQL query.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Can you share your views on mapping description files?

**Answer:** In Hibernate, mapping description files are XML files (commonly named with the .hbm.xml extension) that describe how a Java class maps to a database table. Although modern Hibernate applications often use annotations for mapping, mapping description files are still widely used, especially in legacy systems or when annotations are not preferred.

**Purpose of Mapping Description Files:**

- Define the relationship between Java classes and database tables.

- Map class fields to table columns.

- Specify primary keys, foreign keys, and constraints.

- Describe relationships (one-to-one, one-to-many, many-to-many).

- Provide flexibility to change mapping without modifying source code.

**Example: Mapping File (Student.hbm.xml):**

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <class name="com.example.Student" table="students">
        <id name="id" column="id">
            <generator class="increment"/>
        </id>
        <property name="name" column="name" type="string"/>
        <property name="email" column="email" type="string"/>
    </class>
</hibernate-mapping>
```

And add the mapping to the configuration file:

```xml
<mapping resource="com/example/Student.hbm.xml"/>
```

**Advantages of Mapping Description Files:**

- Separation of concerns: Keeps mapping details out of source code.

- Flexibility: Easy to change mappings without recompiling code.

- Legacy support: Useful for projects that started before annotations existed.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What are your thoughts on file mapping in Hibernate?

**Answer:** File mapping in Hibernate refers to the process of linking Java classes with database tables using an external mapping file (usually .hbm.xml). It is an essential part of Hibernate’s ORM mechanism, especially in scenarios where annotation-based mapping is not desired.

**Key Points on File Mapping:**

1. Decouples Code and Mapping:

- File mapping allows developers to define mapping details externally, enabling them to modify mappings without touching the Java code.

2. Supports Complex Mappings:

- Mapping files can describe advanced relationships like composite keys, embedded objects, and inheritance hierarchies.

3. Useful in Large Projects:

- When multiple developers work on a project, keeping mapping definitions separate helps maintain cleaner and more manageable code.

4. Configuration Flexibility:

- You can switch between databases or modify table structures by simply changing the mapping file.

**Example of File Mapping for Relationships:**

```xml
<class name="com.example.Department" table="departments">
    <id name="id" column="dept_id">
        <generator class="increment"/>
    </id>
    <property name="name" column="dept_name"/>

    <set name="employees" table="employees" inverse="true" cascade="all">
        <key column="dept_id"/>
        <one-to-many class="com.example.Employee"/>
    </set>
</class>
```

**Explanation:**

- The above mapping defines a one-to-many relationship between Department and Employee using a set.

- This mapping is stored externally and can be modified without changing the Java classes.

**Summary:**

- Creating a database application with Hibernate involves configuring hibernate.cfg.xml, defining entity classes, and writing simple Java code to persist data — all without writing SQL queries.

- Mapping description files provide a powerful and flexible way to define how Java objects map to database tables, supporting relationships, keys, and constraints.

- File mapping in Hibernate is especially useful in large projects, legacy systems, and scenarios where developers want to keep mapping logic separate from Java code.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Can you explain version field?

**Answer:** In Hibernate, the version field is a special column used to implement optimistic locking, which helps prevent concurrent modification issues when multiple transactions try to update the same row in the database at the same time.

Optimistic locking works on the assumption that multiple transactions can frequently complete without interfering with each other. However, to detect conflicts when they occur, Hibernate uses a version column that is automatically checked and updated with each modification.

**How It Works:**

- When a record is first inserted, the version column is initialized (typically to 0).

- Each time the entity is updated, Hibernate increments the version number by 1.

- Before committing an update, Hibernate checks if the version number in memory matches the one in the database:

1. ✅ If they match → the update proceeds and the version is incremented.

2. ❌ If they don’t match → Hibernate throws an OptimisticLockException, indicating another transaction modified the entity.

**Example: Using @Version Annotation:**

```java
import javax.persistence.*;

@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int id;

    private String name;
    private String email;

    @Version
    private int version;

    // Getters and Setters
}
```

**Explanation:**

- The @Version annotation marks the version field.

- Hibernate automatically manages this column — you don’t need to update it manually.

- It ensures data consistency and prevents lost updates in concurrent transactions.

**Advantages of Using version:**

- Prevents lost updates in concurrent transactions.

- Ensures data integrity without explicit locking.

- Provides a lightweight concurrency control mechanism.

- Works automatically once configured — no extra logic needed.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What are your views on the function addClass?

**Answer:** The addClass() function in Hibernate is a method used in the Configuration class to register annotated entity classes programmatically with the Hibernate configuration. Instead of defining all mappings in the hibernate.cfg.xml file or using mapping files, you can dynamically add annotated classes using addClass().

##### This approach is useful when:

- You want to configure Hibernate programmatically.

- You don’t want to modify XML configuration files.

- Your application dynamically determines which entities to include.

**Syntax**

```java
Configuration configuration = new Configuration();
configuration.configure("hibernate.cfg.xml");
configuration.addClass(Student.class);
```

**Explanation:**

- configure() loads the configuration from the hibernate.cfg.xml file.

- addClass(Student.class) tells Hibernate that the Student class is an entity it should manage.

**Example: Complete Usage**

```java
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class MainApp {
    public static void main(String[] args) {
        Configuration configuration = new Configuration();
        configuration.configure("hibernate.cfg.xml");
        configuration.addClass(Student.class); // Registering annotated entity class

        SessionFactory sessionFactory = configuration.buildSessionFactory();
        Session session = sessionFactory.openSession();

        Student student = new Student();
        student.setName("Alice");
        student.setEmail("alice@example.com");

        session.beginTransaction();
        session.save(student);
        session.getTransaction().commit();

        session.close();
        sessionFactory.close();
    }
}
```

**Advantages of addClass()**

- Dynamic configuration: Add entity classes at runtime without editing XML.

- Code-centric approach: Keeps configuration within Java code.

- Simplifies deployment: Especially useful in frameworks or tools that generate entity classes dynamically.

**Summary:**

- The version field is essential for implementing optimistic locking in Hibernate. It helps ensure data consistency and prevents concurrent update conflicts by automatically tracking changes to an entity.

- The addClass() function in Hibernate allows developers to programmatically register annotated entity classes, providing a more flexible and dynamic way of configuring Hibernate without relying solely on XML mapping files.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Can you explain the role of addDirectory() and addjar() methods?

**Answer:** In Hibernate, the addDirectory() and addJar() methods are part of the Configuration class and are used to add mapping resources dynamically from specific locations — such as directories or JAR files — to the Hibernate configuration. They provide a flexible and programmatic way to load entity mapping metadata without hardcoding individual class or mapping file references in your configuration.

1. **addDirectory() Method**

The addDirectory() method is used to add a directory containing Hibernate mapping files (.hbm.xml) to the configuration. This is especially useful when you have multiple mapping files organized in a specific folder rather than adding them individually.

**Syntax:**

```java
Configuration cfg = new Configuration();
cfg.configure("hibernate.cfg.xml");
cfg.addDirectory(new File("src/main/resources/mappings"));
```

**Explanation:**

- addDirectory() scans the given directory for all mapping files.

- All .hbm.xml files present in the specified folder are automatically added to the Hibernate configuration.

- This eliminates the need to manually list each mapping file in hibernate.cfg.xml.

**Example: Using addDirectory()**

1. Suppose you have the following directory structure:

src/main/resources/mappings/
│
├── Student.hbm.xml
├── Teacher.hbm.xml
└── Course.hbm.xml

2. Then you can load them all as:

```java
Configuration cfg = new Configuration();
cfg.configure("hibernate.cfg.xml");
cfg.addDirectory(new File("src/main/resources/mappings"));
SessionFactory factory = cfg.buildSessionFactory();
```

#### Result: All three mapping files (Student.hbm.xml, Teacher.hbm.xml, Course.hbm.xml) are registered with Hibernate automatically.

**Benefits of addDirectory()**

- Simplifies the process of adding multiple mapping files.

- Avoids manually specifying each .hbm.xml in hibernate.cfg.xml.

- Improves maintainability and scalability when many mapping files are involved.

2. **addJar() Method**

The addJar() method is used to add a JAR file that contains mapping files or annotated entity classes to the Hibernate configuration. This is particularly useful in modular applications or when mappings are packaged into separate JARs.

**Syntax**

```java
Configuration cfg = new Configuration();
cfg.configure("hibernate.cfg.xml");
cfg.addJar(new File("lib/mappings.jar"));
```

**Explanation:**

- addJar() scans the specified JAR file for any .hbm.xml mapping files or annotated entity classes.

- These mappings are then automatically registered with the Hibernate SessionFactory.

**Comparison: addDirectory() vs addJar()**

| Feature         | `addDirectory()`                                        | `addJar()`                                                         |
| --------------- | ------------------------------------------------------- | ------------------------------------------------------------------ |
| Purpose         | Adds mapping files from a **directory**                 | Adds mapping files or entities from a **JAR file**                 |
| Input           | Directory path (`File` object)                          | JAR file (`File` object)                                           |
| Use Case        | Useful during development when mappings are in a folder | Useful in production or modular setups where mappings are packaged |
| Supported Files | `.hbm.xml` files                                        | `.hbm.xml` files and annotated classes                             |

**Summary**

- addDirectory(): Loads all .hbm.xml mapping files from a given directory, making it easy to manage multiple mappings without editing the configuration file individually.

- addJar(): Loads all mapping files or annotated entities from a JAR file, making it ideal for modular applications or scenarios where mappings are packaged separately.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What do you understand by Hibernate tuning?

**Answer:** Hibernate tuning refers to the process of optimizing the performance and efficiency of Hibernate-based applications. Since Hibernate is an Object-Relational Mapping (ORM) framework that interacts with the database, poor configuration or inefficient queries can lead to performance bottlenecks. Tuning Hibernate ensures that the application runs faster, uses fewer resources, and handles large datasets more effectively.

**Key Techniques for Hibernate Tuning:**

1. Use Lazy Loading Wisely

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;
```

2. Batch Fetching

```xml
<set name="orders" batch-size="20">
    <key column="customer_id"/>
    <one-to-many class="Order"/>
</set>
```

3. Second-Level and Query Caching

```xml
<class-cache usage="read-write" class="com.example.Customer"/>
```

4. Use Pagination for Large Result Sets

```java
Query query = session.createQuery("from Employee");
query.setFirstResult(0);
query.setMaxResults(20);
```

5. Optimize Queries

```java
Query query = session.createQuery("SELECT e.name FROM Employee e");
```

**Summary**

Hibernate tuning is crucial for building scalable, high-performance database applications. By carefully managing how data is loaded, cached, queried, and indexed — and by leveraging features like batch fetching, lazy loading, and caching — you can significantly improve your application’s efficiency and responsiveness.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What is your understanding of Light Object Mapping?

**Answer:** Light Object Mapping in Hibernate refers to a simplified and minimal approach to mapping Java objects (entities) to database tables. It is used when the application requires only basic CRUD operations (Create, Read, Update, Delete) without the need for complex relationships, advanced mappings, or sophisticated configurations.

In other words, Light Object Mapping focuses on mapping simple POJOs (Plain Old Java Objects) to database tables with minimal annotations or XML configurations. It is often preferred in small applications, prototypes, or scenarios where performance and simplicity are more important than flexibility.

**Key Characteristics of Light Object Mapping:**

1. Minimal Configuration:

- Uses straightforward annotations or XML mappings.

- Only essential fields are mapped to database columns.

2. Direct Mapping:

- Each class corresponds to a single database table.

- Each field maps directly to a column.

3. Basic CRUD Operations:

- Focuses on simple data manipulation rather than complex business logic.

4. Fast and Lightweight:

- Since it avoids complex mappings (like inheritance, collections, associations), it’s faster and easier to maintain.

**Example of Light Object Mapping:**

```java
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Student {

    @Id
    private int id;
    private String name;
    private String course;

    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }
}
```

**In this example:**

- Student is a simple POJO mapped to a Student table in the database.

- No complex relationships or advanced features are used.

- Hibernate can easily perform CRUD operations on this entity.

**When to Use Light Object Mapping:**

- For simple applications with minimal database interaction.

- When you don’t need complex relationships (like OneToMany, ManyToMany).

- In prototypes or proof-of-concept projects.

- When performance and simplicity are prioritized over flexibility.

**Summary:**

Light Object Mapping is a minimalistic and straightforward approach in Hibernate, ideal for applications that only require basic database operations. It keeps the code clean, simple, and easy to maintain — making it perfect for small projects or situations where complex ORM features are unnecessary.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How does Hibernate create the database connection?

**Answer:** Hibernate creates a database connection by using the configuration details provided in its configuration file (usually hibernate.cfg.xml) or through Java-based configuration. It uses these settings to build a SessionFactory, which in turn produces Session objects. These Session objects are responsible for interacting with the database.

**In simple terms:**

1. Hibernate reads the database connection properties (like URL, username, password, driver class).

2. It uses these details to configure the JDBC connection internally.

3. A SessionFactory is built from this configuration.

4. From the SessionFactory, Hibernate creates a Session, which acts as a bridge between the application and the database.

**Steps Involved in Database Connection Creation**

1. Load Configuration:
- Hibernate reads the connection details from hibernate.cfg.xml or a Java Configuration class.

2. Build SessionFactory:
- Using the configuration, Hibernate builds a SessionFactory, which is a heavy-weight object and should be created once per application.

3. Open Session:
- A Session is a lightweight object created from the SessionFactory. It represents a single unit of work with the database.

4. Obtain JDBC Connection:
- The Session uses the underlying JDBC connection to execute SQL queries and manage transactions.

**Example: Using hibernate.cfg.xml**

1. Configuration File (hibernate.cfg.xml):

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>

        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/mydb</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">password</property>

        <!-- Hibernate dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</property>

        <!-- Show SQL -->
        <property name="hibernate.show_sql">true</property>

        <!-- Mapped class -->
        <mapping class="com.example.Student"/>

    </session-factory>
</hibernate-configuration>
```

2. Java Code to Create Database Connection:

```java
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateConnectionExample {
    public static void main(String[] args) {
        // Step 1: Load configuration and build SessionFactory
        SessionFactory sessionFactory = new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();

        // Step 2: Open a session
        Session session = sessionFactory.openSession();

        // Step 3: Begin transaction and interact with DB
        session.beginTransaction();

        // Perform database operations here

        session.getTransaction().commit();
        session.close();
        sessionFactory.close();
    }
}
```

**How It Works Internally**

- Hibernate uses the JDBC driver defined in the configuration.

- It manages the connection pooling and transaction handling internally.

- The Session acts as a wrapper around the JDBC connection, allowing developers to work with objects instead of SQL statements.

**Summary:**

Hibernate creates a database connection by reading configuration properties, building a SessionFactory, and generating Session objects that internally manage JDBC connections. This approach abstracts away the complexity of connection management, allowing developers to focus on object manipulation rather than low-level database operations.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. What are possible ways to configure object-table mapping?

**Answer:** In Hibernate, object-table mapping defines how a Java class (object) is linked or mapped to a database table.
This mapping tells Hibernate which table corresponds to which class and how class attributes correspond to table columns.

There are three main ways to configure object-table mapping in Hibernate:

1. XML-Based Mapping (Using .hbm.xml file)

In this traditional method, mappings are defined in an external XML file — typically named ClassName.hbm.xml.
This file explicitly specifies how each class property maps to the database column.

**Example:**

- Student.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD//EN"
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="com.example.Student" table="student">
        <id name="id" column="id">
            <generator class="increment"/>
        </id>
        <property name="name" column="name"/>
        <property name="course" column="course"/>
    </class>
</hibernate-mapping>
```

- Configuration File (hibernate.cfg.xml):

```xml
<mapping resource="com/example/Student.hbm.xml"/>
```

2. Annotation-Based Mapping (Using JPA/Hibernate Annotations)

In this modern approach, mappings are defined directly in the Java class using annotations provided by Hibernate or JPA.
This method makes the code more readable and reduces the need for separate XML files.

- Example:

```java
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Column;

@Entity
@Table(name = "student")
public class Student {

    @Id
    private int id;

    @Column(name = "name")
    private String name;

    @Column(name = "course")
    private String course;

    // Getters and setters
}
```

- Configuration File (hibernate.cfg.xml):

```xml
<mapping class="com.example.Student"/>
```
Annotation-based mapping is the most commonly used method in modern Hibernate and JPA applications.

3. Programmatic (Code-Based) Mapping

Hibernate also supports defining mappings programmatically using the Configuration class or metadata APIs.
This approach is less common but useful when mapping needs to be generated dynamically at runtime.

- Example:

```java
import org.hibernate.cfg.Configuration;

Configuration cfg = new Configuration();
cfg.addAnnotatedClass(Student.class);
cfg.setProperty("hibernate.connection.url", "jdbc:mysql://localhost:3306/mydb");
cfg.setProperty("hibernate.connection.username", "root");
cfg.setProperty("hibernate.connection.password", "password");
```

This approach is ideal when you want flexibility and prefer not to rely on XML or static annotations — for example, in dynamically generated entity mappings.

**Summary:**

| **Method**                   | **Description**                                             | **When to Use**                                    |
| ---------------------------- | ----------------------------------------------------------- | -------------------------------------------------- |
| **XML-Based Mapping**        | Defines mapping externally in `.hbm.xml` files.             | When you want separation between code and mapping. |
| **Annotation-Based Mapping** | Defines mapping directly in Java classes using annotations. | For modern applications (recommended).             |
| **Programmatic Mapping**     | Defines mapping through code using Hibernate APIs.          | When mappings are generated dynamically.           |

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. Which annotation is used to declare a class as a hibernate bean?

**Answer:** In Hibernate (and JPA), the annotation used to declare a class as a Hibernate bean (entity) is:

```java
@Entity
```

**Explanation:**

- The @Entity annotation is used to mark a Java class as a persistent entity, meaning Hibernate will map this class to a corresponding table in the database.

- Once a class is annotated with @Entity, Hibernate treats it as a managed object, capable of being saved, updated, deleted, and retrieved from the database.

**Example:**

```java
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Student {

    @Id
    private int id;
    private String name;
    private String course;

    // Getters and setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }
}
```

**Key Points About @Entity:**

1. The @Entity annotation is part of javax.persistence (JPA specification).

2. The class annotated with @Entity must have a primary key, annotated with @Id.

3. The class must have a default (no-argument) constructor.

4. The class name (by default) is used as the table name — but you can customize it using the @Table annotation.

**Example with Custom Table Name:**

```java
import javax.persistence.Entity;
import javax.persistence.Table;
import javax.persistence.Id;

@Entity
@Table(name = "student_info")
public class Student {

    @Id
    private int id;
    private String name;
}
```
Here, the entity Student is mapped to the table student_info.

**summary:**

| **Annotation** | **Purpose**                                                 |
| -------------- | ----------------------------------------------------------- |
| `@Entity`      | Declares a class as a Hibernate/JPA entity (Hibernate bean) |
| `@Table`       | Specifies the table name if it differs from the class name  |
| `@Id`          | Marks a field as the primary key of the entity              |

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How do I specify table name linked to an entity using annotation?

**Answer:** In Hibernate (and JPA), you can specify the table name linked to an entity using the @Table annotation.
This annotation is used along with the @Entity annotation to define the exact database table that the entity class should map to.

**Syntax:**

```java
@Entity
@Table(name = "table_name")
public class ClassName {
    // fields, constructors, and methods
}
```

- @Entity → Declares the class as a persistent Hibernate entity.

- @Table(name = "table_name") → Specifies the table in the database that this entity is mapped to.

```java
import javax.persistence.Entity;
import javax.persistence.Table;
import javax.persistence.Id;

@Entity
@Table(name = "student_info")
public class Student {

    @Id
    private int id;
    private String name;
    private String course;

    // Getters and setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }
}
```

**In this example:**

- The Java class Student is marked as an entity using @Entity.

- The database table name is explicitly set to student_info using @Table(name = "student_info").

- If the @Table annotation is not specified, Hibernate automatically assumes the table name is the same as the class name (e.g., Student → student).

**Example without @Table:**

```java
@Entity
public class Employee {
    @Id
    private int id;
    private String name;
}
```
Here, Hibernate will assume the table name is Employee by default.

**Additional Attributes of @Table:**

The @Table annotation can also include other optional attributes:

| **Attribute**       | **Description**                                      |
| ------------------- | ---------------------------------------------------- |
| `name`              | Specifies the table name.                            |
| `catalog`           | Defines the catalog of the table.                    |
| `schema`            | Defines the database schema that contains the table. |
| `uniqueConstraints` | Defines unique constraints on columns.               |

**Example:**

```java
@Entity
@Table(
    name = "employee_details",
    schema = "public",
    uniqueConstraints = {@UniqueConstraint(columnNames = {"email"})}
)
public class Employee {
    @Id
    private int id;
    private String email;
}
```

**Summary:**

| **Annotation**                | **Purpose**                                         |
| ----------------------------- | --------------------------------------------------- |
| `@Entity`                     | Declares a class as a Hibernate entity              |
| `@Table(name = "table_name")` | Specifies the exact table name linked to the entity |

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How does a variable in an entity connect to the database column?


**Answer:** In Hibernate, a variable (field) in an entity class is mapped to a column in the database table using either annotations or XML mapping files. This mapping allows Hibernate to know which database column corresponds to which class field, enabling automatic data persistence and retrieval.

1. Using Annotations

The most common approach is to use the @Column annotation along with the @Id annotation for primary keys.

**Syntax**

```java
@Column(name = "column_name")
private DataType variableName;
```

- @Column(name = "column_name") → Maps the variable to a specific column in the table.

- @Id → Marks the variable as the primary key column.

**Example:**

```java
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Column;
import javax.persistence.Table;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @Column(name = "student_id")
    private int id;

    @Column(name = "student_name")
    private String name;

    @Column(name = "student_course")
    private String course;

    // Getters and setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }
}
```

**Explanation:**

- The id field is mapped to the student_id column.

- The name field is mapped to the student_name column.

- The course field is mapped to the student_course column.

- Hibernate automatically manages reading and writing these values to the corresponding database columns.

2. Using XML Mapping (.hbm.xml)

Alternatively, you can define the mapping in an XML file.

```xml
<class name="com.example.Student" table="student">
    <id name="id" column="student_id">
        <generator class="increment"/>
    </id>
    <property name="name" column="student_name"/>
    <property name="course" column="student_course"/>
</class>
```

**Key Points**

1. Each entity variable is mapped to a specific column in the table.

2. The @Column annotation allows you to:

- Specify column name (name)

- Define column length (length)

- Define constraints like nullable, unique

3. If @Column is not specified, Hibernate automatically maps the field name to a column with the same name.

**Summary:**

Variables in an entity connect to database columns through annotations (@Column) or XML mapping files.

Hibernate uses this mapping to persist objects to the database and retrieve data as Java objects, abstracting away SQL queries.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How do we specify a different column name for the variables mapping?

**Answer:** In Hibernate (and JPA), you can specify a different column name for a variable (field) in an entity using the @Column annotation.
By default, Hibernate maps the variable name directly to the column name with the same name, but when you want to customize or override it, you use the name attribute of the @Column annotation.

**Syntax:**

```java
@Column(name = "custom_column_name")
private DataType variableName;
```

name → defines the actual column name in the database table that should be mapped to this field.

**Example:**

```java
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Column;
import javax.persistence.Table;

@Entity
@Table(name = "student")
public class Student {

    @Id
    @Column(name = "student_id")   // Custom column name for ID
    private int id;

    @Column(name = "student_name") // Custom column name for Name
    private String name;

    @Column(name = "student_course") // Custom column name for Course
    private String course;

    // Getters and Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCourse() { return course; }
    public void setCourse(String course) { this.course = course; }
}
```

**Explanation:**

- The Java variable id maps to the database column student_id.

- The variable name maps to student_name.

- The variable course maps to student_course.

This allows Hibernate to understand which exact column each field corresponds to, even if the names differ.

**Without @Column:**

If you do not specify the @Column annotation, Hibernate assumes that the column name is the same as the variable name.

```java
@Entity
public class Student {
    @Id
    private int id;    // Maps to column "id"
    private String name;  // Maps to column "name"
}
```

**Additional @Column Attributes:**

Besides name, the @Column annotation provides several optional attributes to control column behavior:

| **Attribute** | **Description**                                      |
| ------------- | ---------------------------------------------------- |
| `name`        | Specifies the actual column name.                    |
| `nullable`    | Defines whether the column can accept `NULL` values. |
| `unique`      | Ensures the column values are unique.                |
| `length`      | Sets the column length (used for `VARCHAR`).         |
| `updatable`   | Defines whether the column is updatable.             |
| `insertable`  | Defines whether the column is insertable.            |

**Example:**

```java
@Column(name = "email_address", nullable = false, unique = true, length = 100)
private String email;
```

**Summary:**

- To specify a different column name for a variable, use the @Column(name = "custom_name") annotation.

- If not specified, Hibernate uses the variable name as the column name by default.

- @Column also provides additional attributes to control the behavior of the mapped column.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How do we specify a variable to be primary key for the table?

**Answer:** In Hibernate (or JPA), you can specify a variable to be the primary key of the table using the @Id annotation.

**Example:**

```java
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
public class Student {
    
    @Id
    private int studentId;
    private String name;
    private String course;
    
    // Getters and setters
    public int getStudentId() {
        return studentId;
    }
    public void setStudentId(int studentId) {
        this.studentId = studentId;
    }
}
```

**Explanation:**

- @Id → Marks the field as the primary key of the entity (table).
- Hibernate uses this field to uniquely identify each recorded in the database table

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How do you configure the dialect in hibernate.cfg.xml?

**Answer:** In Hibernate, the dialect specifies the SQL variant (flavor) that Hibernate should use to generate database-specific SQL queries.
Each database (like MySQL, Oracle, PostgreSQL, etc.) has its own SQL syntax, and the Hibernate Dialect helps Hibernate understand which SQL commands to use for that particular database.

You configure the dialect in the hibernate.cfg.xml file using the property:

```xml
<property name="hibernate.dialect">dialect_class_name</property>
```

**Example: Configuring Dialect in hibernate.cfg.xml**

Here’s a sample configuration file for a MySQL database:

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>

        <!-- Database connection settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/mydb</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">password</property>

        <!-- Hibernate dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</property>

        <!-- Show SQL in console -->
        <property name="hibernate.show_sql">true</property>

        <!-- Automatically create/update tables -->
        <property name="hibernate.hbm2ddl.auto">update</property>

        <!-- Mapping class -->
        <mapping class="com.example.Student"/>

    </session-factory>
</hibernate-configuration>
```

**Explanation:**

- This property tells Hibernate which database dialect to use.
For example, Hibernate will use MySQL8Dialect for MySQL 8 databases, Oracle12cDialect for Oracle 12c, etc.

```xml
<property name="hibernate.dialect">
```

The dialect helps Hibernate generate database-specific SQL queries automatically.

- Hibernate uses this dialect to:

Generate SQL statements compatible with your database.

Handle differences in data types, keywords, and query syntax between databases.

**Common Hibernate Dialects:**

| **Database** | **Dialect Class**                                          |
| ------------ | ---------------------------------------------------------- |
| MySQL 8      | `org.hibernate.dialect.MySQL8Dialect`                      |
| MySQL 5      | `org.hibernate.dialect.MySQL5Dialect`                      |
| Oracle 12c   | `org.hibernate.dialect.Oracle12cDialect`                   |
| PostgreSQL   | `org.hibernate.dialect.PostgreSQLDialect`                  |
| SQL Server   | `org.hibernate.dialect.SQLServerDialect`                   |
| H2 Database  | `org.hibernate.dialect.H2Dialect`                          |
| SQLite       | `org.hibernate.dialect.SQLiteDialect` (custom/third-party) |

**Why Dialect Is Important ?**

- Different databases have variations in SQL syntax.

- The Hibernate dialect ensures that the generated SQL is compatible with the target database.

- For example, LIMIT in MySQL vs TOP in SQL Server — the dialect handles such differences automatically.

<div align="right">
    <b><a href="#">↥ back to top</a></b>
</div>

## Q. How to configure the database URL and credentials in hibernate.cfg.xml?

## Q. How to configure the connection pool size?

## Q. How do you configure folder scan for Hibernate beans?

## Q. How to configure hibernate beans without Spring framework?

## Q. Is it possible to connect multiple database in a single Java application using Hibernate?

## Q. Does Hibernate support polymorphism?

## Q. How many Hibernate sessions exist at any point of time in an application?

## Q. What is N+1 SELECT problem in Hibernate? What are some strategies to solve the N+1 SELECT problem in Hibernate?

## Q. What is the requirement for a Java object to become a Hibernate entity object?

## Q. How do you log SQL queries issued by the Hibernate framework in Java application?

## Q. What is the difference between the transient, persistent and detached state in Hibernate?

## Q. How properties of a class are mapped to the columns of a database table in Hibernate?

## Q. What is the usage of Configuration Interface in hibernate?

## Q. How can we use new custom interfaces to enhance functionality of built-in interfaces of hibernate?

## Q. What are POJOs and what is their significance?

## Q. How can we invoke stored procedures in hibernate?

## Q. What are the benefits of using Hibernate template?

## Q. How can we get hibernate statistics?

## Q. How can we reduce database write action times in Hibernate?

## Q. When an instance goes in detached state in hibernate?

## Q. What the four ORM levels are in hibernate?

## Q. What is the default cache service of hibernate?

## Q. What are the two mapping associations used in hibernate?

## Q. What is the usage of Hibernate QBC API?

## Q. In how many ways, objects can be fetched from database in hibernate?

## Q. How primary key is created by using hibernate?

## Q. How can we reattach any detached objects in Hibernate?

## Q. What are different ways to disable hibernate second level cache?

## Q. What is ORM metadata?

## Q. Which one is the default transaction factory in hibernate?

## Q. What is the role of JMX in hibernate?

## Q. In how many ways objects can be identified in Hibernate?

## Q. What different fetching strategies are of hibernate?

## Q. How mapping of java objects is done with database tables?

## Q. What are derived properties in hibernate?

## Q. What is the use of version property in hibernate?

## Q. What is attribute oriented programming?

## Q. What is the use of session.lock() in hibernate?

## Q. What the three inheritance models are of hibernate?

## Q. What is general hibernate flow using RDBMS?

## Q. What is difference between managed associations and hibernate associations?

## Q. What are the inheritance mapping strategies?

## Q. What is automatic dirty checking in hibernate?

## Q. Explain Hibernate configuration file and Hibernate mapping file?
