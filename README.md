### Project Setup

1. **Create a Maven Project in NetBeans:**
   - Open NetBeans and create a new Maven Web Application project.
   - Name your project (e.g., `UniversityCourseSelection`).

2. **Add Dependencies in `pom.xml`:**
   ```xml
   <dependencies>
       <dependency>
           <groupId>javax.servlet</groupId>
           <artifactId>javax.servlet-api</artifactId>
           <version>4.0.1</version>
           <scope>provided</scope>
       </dependency>
       <dependency>
           <groupId>javax.servlet.jsp</groupId>
           <artifactId>javax.servlet.jsp-api</artifactId>
           <version>2.3.3</version>
           <scope>provided</scope>
       </dependency>
       <dependency>
           <groupId>com.microsoft.sqlserver</groupId>
           <artifactId>mssql-jdbc</artifactId>
           <version>12.2.0.jre8</version>
       </dependency>
       <dependency>
           <groupId>org.apache.commons</groupId>
           <artifactId>commons-dbcp2</artifactId>
           <version>2.8.0</version>
       </dependency>
       <dependency>
           <groupId>javax.servlet.jsp.jstl</groupId>
           <artifactId>jstl-api</artifactId>
           <version>1.2</version>
       </dependency>
       <dependency>
           <groupId>org.glassfish.web</groupId>
           <artifactId>jstl-impl</artifactId>
           <version>1.2</version>
       </dependency>
   </dependencies>
   ```

3. **Configure Data Source in `context.xml`:**
   - In `src/main/webapp/META-INF/context.xml`:
   ```xml
   <Context>
       <Resource name="jdbc/UniversityDB" auth="Container"
                 type="javax.sql.DataSource" driverClassName="com.microsoft.sqlserver.jdbc.SQLServerDriver"
                 url="jdbc:sqlserver://localhost:1433;databaseName=university_info"
                 username="admin" password="ccitindustry" maxTotal="100" maxIdle="30" maxWaitMillis="10000"/>
   </Context>
   ```

4. **Add Database Configuration in `web.xml`:**
   ```xml
   <resource-ref>
       <description>DB Connection</description>
       <res-ref-name>jdbc/UniversityDB</res-ref-name>
       <res-type>javax.sql.DataSource</res-type>
       <res-auth>Container</res-auth>
   </resource-ref>
   ```

### Database Tables

Ensure your SQL Server database has the required tables:
```sql
CREATE TABLE student_table (
    student_id INT PRIMARY KEY IDENTITY(1,1),
    student_name NVARCHAR(100),
    email NVARCHAR(100),
    username NVARCHAR(50),
    userpassword NVARCHAR(50)
);

CREATE TABLE courses (
    course_id INT PRIMARY KEY IDENTITY(1,1),
    course_name NVARCHAR(100)
);

CREATE TABLE course_selections (
    selection_id INT PRIMARY KEY IDENTITY(1,1),
    student_id INT FOREIGN KEY REFERENCES student_table(student_id),
    username NVARCHAR(50),
    course NVARCHAR(100)
);
```

### Model Classes

1. **Student.java**
   ```java
   public class Student {
       private int studentId;
       private String studentName;
       private String email;
       private String username;
       private String userPassword;

       // Getters and setters
   }
   ```

2. **Course.java**
   ```java
   public class Course {
       private int courseId;
       private String courseName;

       // Getters and setters
   }
   ```

3. **CourseSelection.java**
   ```java
   public class CourseSelection {
       private int selectionId;
       private int studentId;
       private String username;
       private String course;

       // Getters and setters
   }
   ```

### DAO Classes

1. **Database Connection Utility:**
   ```java
   public class DatabaseConnection {
       private static DataSource dataSource;

       static {
           try {
               Context context = new InitialContext();
               dataSource = (DataSource) context.lookup("java:/comp/env/jdbc/UniversityDB");
           } catch (NamingException e) {
               e.printStackTrace();
           }
       }

       public static Connection getConnection() throws SQLException {
           return dataSource.getConnection();
       }
   }
   ```

2. **StudentDAO.java**
   ```java
   public class StudentDAO {
       public List<Student> getAllStudents() {
           List<Student> students = new ArrayList<>();
           String sql = "SELECT * FROM student_table";
           try (Connection conn = DatabaseConnection.getConnection();
                PreparedStatement stmt = conn.prepareStatement(sql);
                ResultSet rs = stmt.executeQuery()) {
               while (rs.next()) {
                   Student student = new Student();
                   student.setStudentId(rs.getInt("student_id"));
                   student.setStudentName(rs.getString("student_name"));
                   student.setEmail(rs.getString("email"));
                   student.setUsername(rs.getString("username"));
                   student.setUserPassword(rs.getString("userpassword"));
                   students.add(student);
               }
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return students;
       }

       // Add other CRUD methods (create, update, delete)
   }
   ```

3. **CourseDAO.java**
   ```java
   public class CourseDAO {
       public List<Course> getAllCourses() {
           List<Course> courses = new ArrayList<>();
           String sql = "SELECT * FROM courses";
           try (Connection conn = DatabaseConnection.getConnection();
                PreparedStatement stmt = conn.prepareStatement(sql);
                ResultSet rs = stmt.executeQuery()) {
               while (rs.next()) {
                   Course course = new Course();
                   course.setCourseId(rs.getInt("course_id"));
                   course.setCourseName(rs.getString("course_name"));
                   courses.add(course);
               }
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return courses;
       }

       // Add other CRUD methods (create, update, delete)
   }
   ```

4. **CourseSelectionDAO.java**
   ```java
   public class CourseSelectionDAO {
       public List<CourseSelection> getAllSelections() {
           List<CourseSelection> selections = new ArrayList<>();
           String sql = "SELECT * FROM course_selections";
           try (Connection conn = DatabaseConnection.getConnection();
                PreparedStatement stmt = conn.prepareStatement(sql);
                ResultSet rs = stmt.executeQuery()) {
               while (rs.next()) {
                   CourseSelection selection = new CourseSelection();
                   selection.setSelectionId(rs.getInt("selection_id"));
                   selection.setStudentId(rs.getInt("student_id"));
                   selection.setUsername(rs.getString("username"));
                   selection.setCourse(rs.getString("course"));
                   selections.add(selection);
               }
           } catch (SQLException e) {
               e.printStackTrace();
           }
           return selections;
       }

       // Add other CRUD methods (create, update, delete)
   }
   ```

### Servlet and JSP

1. **Servlet for Students:**
   ```java
   @WebServlet("/students")
   public class StudentServlet extends HttpServlet {
       private StudentDAO studentDAO;

       public void init() {
           studentDAO = new StudentDAO();
       }

       protected void doGet(HttpServletRequest request, HttpServletResponse response)
               throws ServletException, IOException {
           List<Student> students = studentDAO.getAllStudents();
           request.setAttribute("students", students);
           RequestDispatcher dispatcher = request.getRequestDispatcher("student-list.jsp");
           dispatcher.forward(request, response);
       }

       // Add doPost for create, update, delete
   }
   ```

2. **JSP for Student List:**
   ```jsp
   <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
   <!DOCTYPE html>
   <html>
   <head>
       <title>Student List</title>
   </head>
   <body>
       <h1>Students</h1>
       <table border="1">
           <tr>
               <th>ID</th>
               <th>Name</th>
               <th>Email</th>
               <th>Username</th>
               <th>Password</th>
           </tr>
           <c:forEach var="student" items="${students}">
               <tr>
                   <td>${student.studentId}</td>
                   <td>${student.studentName}</td>
                   <td>${student.email}</td>
                   <td>${student.username}</td>
                   <td>${student.userPassword}</td>
               </tr>
           </c:forEach>
       </table>
   </body>
   </html>
   ```

3. **Repeat similar steps for `CourseServlet`, `CourseSelectionServlet`, and their corresponding JSP files.**

### MVC Structure

- **Model:** Contains your entity classes and DAO classes.
- **View:** Contains your JSP files.
- **Controller:** Contains your Servlet classes.

### Running the Application

1. **Deploy the application:**
   - Right-click on the project and select `Run`.

2. **Access the application:**
   - Open your browser and navigate to `http://localhost:8080/UniversityCourseSelection/students` to see the list of students.
