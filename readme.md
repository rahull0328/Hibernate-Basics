# Hibernate Basics

> *Click &#9733; if you like the project. Your contributions are heartily ♡ welcome.*

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

* **spring-boot-starter-data-jpa**(required): It includes spring data, hibernate, HikariCP, JPA API, JPA Implementation (default is hibernate), JDBC and other required libraries.

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

## Q. Mention the differences between JPA and Hibernate?

JPA is a specification for accessing, persisting and managing the data between Java objects and the relational database. 

Where as, Hibernate is the actual implementation of JPA guidelines. When hibernate implements the JPA specification, this will be certified by the JPA group upon following all the standards mentioned in the specification. For example, JPA guidelines would provide information of mandatory and optional features to be implemented as part of the JPA implementation.

|Hibernate                                  |JPA                                                  |
|-------------------------------------------|-----------------------------------------------------|
|Hibernate is the object-relational mapping framework which helps to deal with the data persistence.|It is the Java specification to manage the java application with relational data.|
|It’s is one of the best JPA providers.	    |It is the only specification which doesn’t deal with any implementation.|
|In this, we use Session for handling the persistence in an application.|In this, we use the Entity manager.         |
|It is used to map Java data types with database tables and SQL data types.|It is the standard API which allows developers to perform database operations smoothly.|
|The Query language in this is Hibernate Query Language.|The query language of JPA is JPQL (Java Persistence Query Language)|

## Q. What is HQL and what are its benefits?

Hibernate Query Language (HQL) is an object-oriented query language, similar to SQL, but instead of operating on tables and columns, HQL works with persistent objects and their properties. HQL queries are translated by Hibernate into conventional SQL queries, which in turns perform action on database.

**Advantages Of HQL:**  

* HQL is database independent, means if we write any program using HQL commands then our program will be able to execute in all the databases with out doing any further changes to it
* HQL supports object oriented features like Inheritance, polymorphism,Associations(Relation ships)
* HQL is initially given for selecting object from database and in hibernate 3.x we can doDML operations ( insert, update…) too

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

* There will be a session factory per database.
* The SessionFacory is built once at start-up
* It is a thread safe class
* SessionFactory will create a new Session object when requested
**Session**:  
* The Session object will get physical connection to the database.
* Session is the Java object used for any DB operations.
* Session is not thread safe. Hence do not share hibernate session between threads
* Session represents unit of work with database
* Session should be closed once the task is completed

## Q. Explain Session object in Hibernate?

A Session is used to get a physical connection with a database. The Session object is lightweight and designed to be instantiated each time an interaction is needed with the database. Persistent objects are saved and retrieved through a Session object.

The lifecycle of a Session is bounded by the beginning and end of a logical transaction. The main function of the Session is to offer create, read and delete operations for instances of mapped entity classes. Instances may exist in one of three states:

* **transient** − A new instance of a persistent class, which is not associated with a Session and has no representation in the database and no identifier value is considered transient by Hibernate.

* **persistent** − You can make a transient instance persistent by associating it with a Session. A persistent instance has a representation in the database, an identifier value and is associated with a Session.

* **detached** − Once we close the Hibernate Session, the persistent instance will become a detached instance.

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


|Transaction interface      | Description              |
|---------------------------|--------------------------|
|void begin()               |starts a new transaction. |
|void commit()              |ends the unit of work unless we are in FlushMode.NEVER.|
|void rollback()            |forces this transaction to rollback.|
|void setTimeout(int seconds)| It sets a transaction timeout for any transaction started by a subsequent call to begin on this |instance.|
|boolean isAlive()          |checks if the transaction is still alive.|
|void registerSynchronization(Synchronization s) |registers a user synchronization callback for this transaction.|
|boolean wasCommited()      |checks if the transaction is commited successfully.|
|boolean wasRolledBack()    |checks if the transaction is rolledback successfully.|

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