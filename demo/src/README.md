# SpringApp 🐾 A Simple User Management App

Welcome to the User Management App! 🎉 This delightful little project allows you to easily manage user information, including their names and emails. It's built with love using Spring Boot and PostgreSQL.

## 🌟 Features

- **User Registration**: Add new users to the system with just a few clicks! 📝
- **User List**: View all registered users in a neat table format. 📋
- **Thymeleaf Integration**: Beautifully rendered views using Thymeleaf for a smooth user experience! 🌈

## 🚀 Getting Started Locally

### Prerequisites

Before you begin, make sure you have the following installed on your local machine:

- [Java 17](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)
- [Maven](https://maven.apache.org/download.cgi)
- [PostgreSQL](https://www.postgresql.org/download/)

### Setup PostgreSQL Database

1. **Create a Database**: Create a new database in PostgreSQL:

   ```sql
   CREATE DATABASE springbootdb;
   ```

2. **Update `application.properties`**: Modify the `src/main/resources/application.properties` file to configure your database connection:

   ```properties
   spring.datasource.url=jdbc:postgresql://localhost:5432/springbootdb
   spring.datasource.username=your_username
   spring.datasource.password=your_password
   ```

### Running the Application Locally

To run the application, follow these steps:

1. **Clean and Build the Project**: Open your terminal, navigate to the project directory, and run:

   ```bash
   ./mvnw clean spring-boot:run
   ```

2. **Access the Application**: Open your web browser and go to [http://localhost:8080/users](http://localhost:8080/users) to see your user management interface! 🌍

### Adding Users with Postman

You can easily add new users using Postman or any other API testing tool:

1. **POST Request** to add a new user:

   - **URL**: `http://localhost:8080/users`
   - **Body** (JSON):
     ```json
     {
       "name": "John Doe",
       "email": "johndoe@example.com"
     }
     ```
