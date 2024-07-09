### 1. User Authentication (Login)

#### Servlet: `LoginServlet.java`
```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    private StudentDAO studentDAO;

    public void init() {
        studentDAO = new StudentDAO();
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        
        Student student = studentDAO.validateStudent(username, password);
        if (student != null) {
            request.getSession().setAttribute("student", student);
            response.sendRedirect("dashboard.jsp");
        } else {
            request.setAttribute("errorMessage", "Invalid Username or Password");
            RequestDispatcher dispatcher = request.getRequestDispatcher("login.jsp");
            dispatcher.forward(request, response);
        }
    }
}
```

#### JSP: `login.jsp`
```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="login" method="post">
        <label for="username">Username:</label>
        <input type="text" name="username" required><br>
        <label for="password">Password:</label>
        <input type="password" name="password" required><br>
        <button type="submit">Login</button>
    </form>
    <c:if test="${not empty errorMessage}">
        <p style="color: red;">${errorMessage}</p>
    </c:if>
</body>
</html>
```

#### DAO Method: `StudentDAO.java`
```java
public class StudentDAO {
    public Student validateStudent(String username, String password) {
        String sql = "SELECT * FROM student_table WHERE username = ? AND userpassword = ?";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setString(1, username);
            stmt.setString(2, password);
            ResultSet rs = stmt.executeQuery();
            if (rs.next()) {
                Student student = new Student();
                student.setStudentId(rs.getInt("student_id"));
                student.setStudentName(rs.getString("student_name"));
                student.setEmail(rs.getString("email"));
                student.setUsername(rs.getString("username"));
                student.setUserPassword(rs.getString("userpassword"));
                return student;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

### 2. Choose Course

#### Servlet: `CourseSelectionServlet.java`
```java
@WebServlet("/chooseCourse")
public class CourseSelectionServlet extends HttpServlet {
    private CourseDAO courseDAO;
    private CourseSelectionDAO courseSelectionDAO;

    public void init() {
        courseDAO = new CourseDAO();
        courseSelectionDAO = new CourseSelectionDAO();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        List<Course> courses = courseDAO.getAllCourses();
        request.setAttribute("courses", courses);
        RequestDispatcher dispatcher = request.getRequestDispatcher("choose-course.jsp");
        dispatcher.forward(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        int studentId = Integer.parseInt(request.getParameter("studentId"));
        String username = request.getParameter("username");
        String course = request.getParameter("course");

        CourseSelection selection = new CourseSelection();
        selection.setStudentId(studentId);
        selection.setUsername(username);
        selection.setCourse(course);

        courseSelectionDAO.saveCourseSelection(selection);
        response.sendRedirect("dashboard.jsp");
    }
}
```

#### JSP: `choose-course.jsp`
```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Choose Course</title>
</head>
<body>
    <h1>Choose Course</h1>
    <form action="chooseCourse" method="post">
        <input type="hidden" name="studentId" value="${sessionScope.student.studentId}">
        <input type="hidden" name="username" value="${sessionScope.student.username}">
        <label for="course">Select Course:</label>
        <select name="course">
            <c:forEach var="course" items="${courses}">
                <option value="${course.courseName}">${course.courseName}</option>
            </c:forEach>
        </select><br>
        <button type="submit">Choose</button>
    </form>
</body>
</html>
```

#### DAO Method: `CourseSelectionDAO.java`
```java
public class CourseSelectionDAO {
    public void saveCourseSelection(CourseSelection selection) {
        String sql = "INSERT INTO course_selections (student_id, username, course) VALUES (?, ?, ?)";
        try (Connection conn = DatabaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            stmt.setInt(1, selection.getStudentId());
            stmt.setString(2, selection.getUsername());
            stmt.setString(3, selection.getCourse());
            stmt.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

### 3. List Courses

#### Servlet: `CourseListServlet.java`
```java
@WebServlet("/courseList")
public class CourseListServlet extends HttpServlet {
    private CourseDAO courseDAO;

    public void init() {
        courseDAO = new CourseDAO();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        List<Course> courses = courseDAO.getAllCourses();
        request.setAttribute("courses", courses);
        RequestDispatcher dispatcher = request.getRequestDispatcher("course-list.jsp");
        dispatcher.forward(request, response);
    }
}
```

#### JSP: `course-list.jsp`
```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Course List</title>
</head>
<body>
    <h1>Course List</h1>
    <table border="1">
        <tr>
            <th>Course ID</th>
            <th>Course Name</th>
        </tr>
        <c:forEach var="course" items="${courses}">
            <tr>
                <td>${course.courseId}</td>
                <td>${course.courseName}</td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```

### 4. List Students Corresponding to Their Course

#### Servlet: `StudentCourseListServlet.java`
```java
@WebServlet("/studentCourseList")
public class StudentCourseListServlet extends HttpServlet {
    private CourseSelectionDAO courseSelectionDAO;

    public void init() {
        courseSelectionDAO = new CourseSelectionDAO();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        List<CourseSelection> selections = courseSelectionDAO.getAllSelections();
        request.setAttribute("selections", selections);
        RequestDispatcher dispatcher = request.getRequestDispatcher("student-course-list.jsp");
        dispatcher.forward(request, response);
    }
}
```

#### JSP: `student-course-list.jsp`
```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Student Course List</title>
</head>
<body>
    <h1>Student Course List</h1>
    <table border="1">
        <tr>
            <th>Student ID</th>
            <th>Username</th>
            <th>Course</th>
        </tr>
        <c:forEach var="selection" items="${selections}">
            <tr>
                <td>${selection.studentId}</td>
                <td>${selection.username}</td>
                <td>${selection.course}</td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```

### 5. Dashboard (Main Menu)

#### JSP: `dashboard.jsp`
```jsp
<!DOCTYPE html>
<html>
<head>
    <title>Dashboard</title>
</head>
<body>
    <h1>Welcome, ${sessionScope.student.studentName}</h1>
    <nav>
        <ul>
            <li><a href="courseList">List Courses</a></li>
            <li><a href="studentCourseList">List Students by Course</a></li>
            <li><a href="chooseCourse">Choose Course</a></li>
            <li><a href="students">List Students</a></li>
            <li><a href="logout">Logout</a></li>
        </ul>
    </nav>
</body>
</html>
```

### 6. Logout

#### Servlet: `LogoutServlet.java`
```java
@WebServlet("/logout")
public class LogoutServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        request.getSession().invalidate();
        response.sendRedirect("login.jsp");
    }
}
```
