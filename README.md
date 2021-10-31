# spring_boot_rest_api_with_jpa_and_hibernate
Making a rest api service using mysql database, spring-data-jpa

### Introduction

- In this project we are going to see how to make a rest api service in spring boot.
- We will see various approaches of how to implement CRUD operations.

### Preparing the project - doing basic work

#### STEP 1 : Adding dependencies

	<dependencies>
  
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
    
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
    
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
    
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
    
	</dependencies>

- spring-boot-starter-data-jpa
- mysql-connector-java
- spring-boot-starter-web
- spring-boot-devtools

#### STEP 2 : Configuring application.properties

    server.port=4522
    
    #JDBC properties
    spring.datasource.url=jdbc:mysql://localhost:3306/employee_directory?useSSL=false&serverTimezone=UTC
    spring.datasource.username=username
    spring.datasource.password=password

#### STEP 3 : Adding Entity class - Employee.java

##### Employee.java

```java
package com.rohitThebest.springboot.cruddemo.entity;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "employee")
public class Employee {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int id;
	
	@Column(name = "first_name")
	private String firstName;
	
	@Column(name = "last_name")
	private String lastName;
	
	@Column(name = "email")
	private String email;

	public Employee() {
	}
	
	public Employee(String firstName, String lastName, String email) {
		super();
		this.firstName = firstName;
		this.lastName = lastName;
		this.email = email;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

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

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	@Override
	public String toString() {
		return "Employee [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email + "]";
	}
}
```

- That's all with our basic preperations.

---

## Using Hibernate with Service

- By default, spring boot gives us *EntityManager* object which we can use for getting the *Session* object and use it for database operations.

#### STEP 1 : Make a *DAO* interface

##### EmployeeDAO.java

```java
package com.rohitThebest.springboot.cruddemo.dao;

import java.util.List;

import com.rohitThebest.springboot.cruddemo.entity.Employee;

public interface EmployeeDAO {

	public List<Employee> findAll();
	
	public Employee findById(int id);
	
	public void save(Employee employee);
	
	public void delete(int id);
}
```

#### STEP 2 : Make the DAO inplmentation class

##### EmployeeDAOHibernateImpl.java

```java
package com.rohitThebest.springboot.cruddemo.dao;

import java.util.List;

import javax.persistence.EntityManager;

import org.hibernate.Session;
import org.hibernate.query.Query;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import com.rohitThebest.springboot.cruddemo.entity.Employee;

@Repository
public class EmployeeDAOHibernateImpl implements EmployeeDAO {

	// define field for entity manager
	private EntityManager entityManager;

	// set up constructor injection
	/*
	 * This entity manager is automatically created by the spring
	 */
	@Autowired
	public EmployeeDAOHibernateImpl(EntityManager entityManager) {
		this.entityManager = entityManager;
	}

	@Override
	public List<Employee> findAll() {

		// get the current hibernate session
		Session currentSession = entityManager.unwrap(Session.class);

		// create a query
		Query<Employee> query = 
				currentSession.createQuery("from Employee", Employee.class);

		// execute query and get the result list
		List<Employee> employees = query.getResultList();
		
		// return the results
		return employees;
	}

	@Override
	public Employee findById(int id) {

		Session currentSession = entityManager.unwrap(Session.class);

		Employee employee =
				currentSession.get(Employee.class, id);
		
		return employee;
	}

	@Override
	public void save(Employee employee) {
	
		Session currentSession = entityManager.unwrap(Session.class);

		currentSession.saveOrUpdate(employee);
	}

	@Override
	public void delete(int id) {

		Session currentSession = entityManager.unwrap(Session.class);

		Query query = currentSession.createQuery(
				"delete from Employee where id=:employeeId");
		
		query.setParameter("employeeId", id);
		
		query.executeUpdate();
	}
}

```

- Here we used the EntityManager object injected by the spring and using it to get the *Session* object.
- And therefore using the *Session* to interact with database.


#### STEP 3 : Make a service interface

##### EmployeeService.java

```java

package com.rohitThebest.springboot.cruddemo.service;

import java.util.List;

import com.rohitThebest.springboot.cruddemo.entity.Employee;

public interface EmployeeService {

	public List<Employee> findAll();

	public Employee findById(int id);

	public void save(Employee employee);

	public void delete(int id);
}

```

#### STEP 4 : Make an inplementstion for service interface

##### EmplyeeServiceImpl.java

```java

package com.rohitThebest.springboot.cruddemo.service;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.rohitThebest.springboot.cruddemo.dao.EmployeeDAO;
import com.rohitThebest.springboot.cruddemo.entity.Employee;

@Service
public class EmplyeeServiceImpl implements EmployeeService {

	@Autowired
	private EmployeeDAO employeeDAO;
	
	@Override
	@Transactional
	public List<Employee> findAll() {
		
		return employeeDAO.findAll();
	}

	@Override
	@Transactional
	public Employee findById(int id) {
		return employeeDAO.findById(id);
	}

	@Override
	@Transactional
	public void save(Employee employee) {
		
		employeeDAO.save(employee);
	}

	@Override
	@Transactional
	public void delete(int id) {

		employeeDAO.delete(id);
	}

}

```

#### STEP 5 : Finally make the controller class

##### EmployeeRestController.java

```java

package com.rohitThebest.springboot.cruddemo.rest;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.rohitThebest.springboot.cruddemo.entity.Employee;
import com.rohitThebest.springboot.cruddemo.service.EmployeeService;

@RestController
@RequestMapping("/api")
public class EmployeeRestController {

	// inject employee dao
	@Autowired
	private EmployeeService employeeService;
	
	// expose "/employees" and return list of employees
	@GetMapping("/employees")
	public List<Employee> findAll() {
		
		return employeeService.findAll();
	}
	
	@GetMapping("/employees/{employeeId}")
	public Employee getEmployee(@PathVariable int employeeId) {
		
		Employee employee = employeeService.findById(employeeId);
		
		if (employee == null) {
			
			throw new RuntimeException("Employee id not found - " + employeeId);
			
		}
		
		return employee;
	}
	
	@PostMapping("/employees")
	public Employee addEmployee(@RequestBody Employee employee) {
		
		// also just in case they pass an id in JSON ... set od to 0
		// this is to force a save of new item... intead of Update
		
		employee.setId(0);
		
		employeeService.save(employee);
		
		return employee;
		
	}
	
	@PutMapping("/employees")
	public Employee updateEmployee(@RequestBody Employee employee) {
		
		employeeService.save(employee);
		
		return employee;
	}
	
	@DeleteMapping("/employees/{employeeId}")
	public String deleteEmployee(@PathVariable int employeeId) {
		
		Employee employee = employeeService.findById(employeeId);
		
		if (employee == null) {
			
			throw new RuntimeException("Employee with id " + employeeId + " does not exist");
		}
		
		employeeService.delete(employeeId);
		
		return "Deleted employee with id : " + employeeId;
		
	}
}

```

---

## Using JPA

- We need to make slight modifications to our app for using the JPA

#### STEP 1 : Delete the previous DAO implementation class i.e. EmployeeDAOHibernateImpl.java

#### STEP 2 : Make a new DAO implementation class, EmployeeDAOJpaImpl.java

##### EmployeeDAOJpaImpl.java

```java
package com.rohitThebest.springboot.cruddemo.dao;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.Query;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.rohitThebest.springboot.cruddemo.entity.Employee;

@Repository
public class EmployeeDAOJpaImpl implements EmployeeDAO {

	@Autowired
	private EntityManager entityManager;

	@Override
	public List<Employee> findAll() {

		// create a query
		Query query = entityManager.createQuery("from Employee");

		// execute query and get result list
		List<Employee> employees = query.getResultList();

		// return the results
		return employees;
	}

	@Override
	public Employee findById(int id) {
		
		Employee employee = entityManager.find(Employee.class, id);
		
		return employee;
	}

	@Override
	public void save(Employee employee) {
		
		// will save or update (if id = 0 -> save otherwise update)
		Employee dbEmployee = entityManager.merge(employee);
		
		// update with id from db... so we can get generated id for save/ insert
		employee.setId(dbEmployee.getId());
	}

	@Override
	public void delete(int id) {
		
		// delete object with primary key
		
		Query query = entityManager.createQuery(
				"delete from Employee where id=:employeeId"
				);
		
		query.setParameter("employeeId", id);
		
		query.executeUpdate();
	}

}

```

- That's all we need, everything remains unchanged.

---

## Using Spring Data JPA

- Now we are going to see a big change.
- When using Spring Data JPA,
  - We don't have to write the implmentations for basic crud opeartions we were doing in *EmployeeDAOImpl*.

#### STEP 1 : Delete the EmployeeDao.java interface and EmployeeDAOJpaImpl.java classes

#### STEP 2 : Make an interface *EmployeeRepository.java* and extend it with *JpaRepository<Employee, Integer>*

- *JPARepository* : is class which contains all basic database operations, we just need to pass the Entity class and the type of primary key.

##### EmployeeRepository.java

```java

package com.rohitThebest.springboot.cruddemo.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.rohitThebest.springboot.cruddemo.entity.Employee;

public interface EmployeeRepository extends JpaRepository<Employee, Integer> {

}

```

#### STEP 3 : Update the *EmplyeeServiceImpl.java* class to use EmployeeRepository

##### EmplyeeServiceImpl.java

```java

package com.rohitThebest.springboot.cruddemo.service;

import java.util.List;
import java.util.Optional;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.rohitThebest.springboot.cruddemo.dao.EmployeeRepository;
import com.rohitThebest.springboot.cruddemo.entity.Employee;

@Service
public class EmplyeeServiceImpl implements EmployeeService {

	@Autowired
	private EmployeeRepository employeeRepository;

	@Override
	public List<Employee> findAll() {

		return employeeRepository.findAll();
	}

	@Override
	public Employee findById(int id) {

		Optional<Employee> result = employeeRepository.findById(id);

		Employee employee = null;

		if (result.isPresent()) {

			employee = result.get();
		} else {

			throw new RuntimeException("Did not find employee id : " + id);
		}

		return employee;
	}

	@Override
	public void save(Employee employee) {

		employeeRepository.save(employee);
	}

	@Override
	public void delete(int id) {

		employeeRepository.deleteById(id);
	}

}

```

- That's it...
- Spring Data JPA, really makes the job very easy and helps in reduction of codes.

