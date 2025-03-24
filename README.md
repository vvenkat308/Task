# Task
The Task Management Application is a microservices-based system built using Java 8+ and Spring Boot. It provides task creation, assignment, and tracking features. The backend ensures secure authentication using JWT-based RBAC (Role-Based Access Control) and integrates seamlessly with CI/CD pipelines for automated deployments using Jenkins, Docker, and Kubernetes.

This application ensures scalability through containerized deployments and maintains code quality and security with SonarQube and Black Duck Scanning.

 How to Code
Below is a step-by-step breakdown of coding the backend for this application.

1️⃣ Set Up Spring Boot Project
Use Spring Initializr to generate a Spring Boot project with the following dependencies:

Spring Web (for building RESTful APIs)

Spring Security (for authentication)

Spring Data JPA (for database integration)

MySQL Driver (or MongoDB if using NoSQL)

Lombok (to reduce boilerplate code)

JWT (jjwt) (for authentication)


2️⃣ Configure Database & Application Properties
Inside src/main/resources/application.properties, configure your database connection:

properties
Copy
Edit
server.port=8080

# MySQL Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/task_db
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA Hibernate Configuration
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

3️⃣ Create Models (Entities)
Inside src/main/java/com/example/taskmanagement/models/, define an entity:

java
Copy
Edit
@Entity
@Table(name = "tasks")
@Data  // Lombok to generate getters & setters
public class Task {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String description;
    private String status;
}

4️⃣ Implement JWT Authentication
Create a JwtUtil class inside src/main/java/com/example/taskmanagement/security/:

java
Copy
Edit
@Component
public class JwtUtil {
    private final String SECRET_KEY = "your-secret-key";

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
    }

    public String extractUsername(String token) {
        return Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateToken(String token, UserDetails userDetails) {
        return extractUsername(token).equals(userDetails.getUsername());
    }
}

5️⃣ Build RESTful API Controllers
Inside src/main/java/com/example/taskmanagement/controllers/, create TaskController.java:

java
Copy
Edit
@RestController
@RequestMapping("/tasks")
public class TaskController {
    @Autowired
    private TaskService taskService;

    @PostMapping("/create")
    public ResponseEntity<Task> createTask(@RequestBody Task task) {
        return ResponseEntity.ok(taskService.createTask(task));
    }

    @GetMapping("/{id}")
    public ResponseEntity<Task> getTask(@PathVariable Long id) {
        return ResponseEntity.ok(taskService.getTaskById(id));
    }
}


6️⃣ CI/CD Pipeline with Jenkins
Create a Jenkinsfile for automation:

groovy
Copy
Edit
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t myrepo/task-management:latest .'
                sh 'docker push myrepo/task-management:latest'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
            }
        }
    }
}

7️⃣ Dockerize the Application
Create a Dockerfile:

dockerfile
Copy
Edit
FROM openjdk:17
WORKDIR /app
COPY target/task-management.jar task-management.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "task-management.jar"]


8️⃣ Deploy to Kubernetes
Create deployment.yaml for Kubernetes:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: task-management
spec:
  replicas: 2
  selector:
    matchLabels:
      app: task-management
  template:
    metadata:
      labels:
        app: task-management
    spec:
      containers:
        - name: task-management
          image: myrepo/task-management:latest
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: task-management-service
spec:
  selector:
    app: task-management
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer

This guide includes:

Spring Boot setup with JWT-based authentication.

RESTful API for managing tasks.

Docker containerization for deployment.

CI/CD pipeline with Jenkins.

Kubernetes deployment for scalability.
