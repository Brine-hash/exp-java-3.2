import java.sql.*;
import java.util.*;

// ========================= PART (a): BASIC JDBC CONNECTION =========================
class JDBCConnectionDemo {
    public static void fetchEmployees() {
        String url = "jdbc:mysql://localhost:3306/testdb"; // change database name
        String user = "root"; // change username
        String pass = "password"; // change password

        try (Connection con = DriverManager.getConnection(url, user, pass);
             Statement stmt = con.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM Employee")) {

            System.out.println("Connected Successfully! Employee Table Data:\n");
            while (rs.next()) {
                System.out.println("EmpID: " + rs.getInt("EmpID") +
                        ", Name: " + rs.getString("Name") +
                        ", Salary: " + rs.getDouble("Salary"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// ========================= PART (b): CRUD OPERATIONS ON PRODUCT =========================
class ProductCRUD {
    private static final String URL = "jdbc:mysql://localhost:3306/testdb";
    private static final String USER = "root";
    private static final String PASS = "password";

    public static void manageProducts() {
        try (Connection con = DriverManager.getConnection(URL, USER, PASS);
             Scanner sc = new Scanner(System.in)) {

            con.setAutoCommit(false);
            boolean exit = false;

            while (!exit) {
                System.out.println("\n=== PRODUCT MANAGEMENT MENU ===");
                System.out.println("1. Insert Product");
                System.out.println("2. View All Products");
                System.out.println("3. Update Product");
                System.out.println("4. Delete Product");
                System.out.println("5. Exit");
                System.out.print("Enter choice: ");
                int ch = sc.nextInt();

                switch (ch) {
                    case 1 -> insertProduct(con, sc);
                    case 2 -> viewProducts(con);
                    case 3 -> updateProduct(con, sc);
                    case 4 -> deleteProduct(con, sc);
                    case 5 -> exit = true;
                    default -> System.out.println("Invalid choice!");
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void insertProduct(Connection con, Scanner sc) {
        String sql = "INSERT INTO Product (ProductID, ProductName, Price, Quantity) VALUES (?, ?, ?, ?)";
        try (PreparedStatement ps = con.prepareStatement(sql)) {
            System.out.print("Enter Product ID: ");
            ps.setInt(1, sc.nextInt());
            sc.nextLine();
            System.out.print("Enter Product Name: ");
            ps.setString(2, sc.nextLine());
            System.out.print("Enter Price: ");
            ps.setDouble(3, sc.nextDouble());
            System.out.print("Enter Quantity: ");
            ps.setInt(4, sc.nextInt());

            ps.executeUpdate();
            con.commit();
            System.out.println("Product Inserted Successfully!");
        } catch (Exception e) {
            try { con.rollback(); } catch (SQLException ex) {}
            System.out.println("Error inserting product!");
        }
    }

    private static void viewProducts(Connection con) {
        String sql = "SELECT * FROM Product";
        try (Statement st = con.createStatement(); ResultSet rs = st.executeQuery(sql)) {
            System.out.println("\nProductID | ProductName | Price | Quantity");
            while (rs.next()) {
                System.out.println(rs.getInt("ProductID") + " | " +
                        rs.getString("ProductName") + " | " +
                        rs.getDouble("Price") + " | " +
                        rs.getInt("Quantity"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void updateProduct(Connection con, Scanner sc) {
        String sql = "UPDATE Product SET Price=?, Quantity=? WHERE ProductID=?";
        try (PreparedStatement ps = con.prepareStatement(sql)) {
            System.out.print("Enter Product ID to Update: ");
            int id = sc.nextInt();
            System.out.print("Enter New Price: ");
            double price = sc.nextDouble();
            System.out.print("Enter New Quantity: ");
            int qty = sc.nextInt();

            ps.setDouble(1, price);
            ps.setInt(2, qty);
            ps.setInt(3, id);

            int rows = ps.executeUpdate();
            if (rows > 0) {
                con.commit();
                System.out.println("Product Updated Successfully!");
            } else System.out.println("No Product Found!");
        } catch (Exception e) {
            try { con.rollback(); } catch (SQLException ex) {}
            System.out.println("Error updating product!");
        }
    }

    private static void deleteProduct(Connection con, Scanner sc) {
        String sql = "DELETE FROM Product WHERE ProductID=?";
        try (PreparedStatement ps = con.prepareStatement(sql)) {
            System.out.print("Enter Product ID to Delete: ");
            int id = sc.nextInt();
            ps.setInt(1, id);

            int rows = ps.executeUpdate();
            if (rows > 0) {
                con.commit();
                System.out.println("Product Deleted Successfully!");
            } else System.out.println("No Product Found!");
        } catch (Exception e) {
            try { con.rollback(); } catch (SQLException ex) {}
            System.out.println("Error deleting product!");
        }
    }
}

// ========================= PART (c): STUDENT MANAGEMENT (MVC) =========================
class Student {
    private int studentID;
    private String name;
    private String department;
    private double marks;

    public Student(int studentID, String name, String department, double marks) {
        this.studentID = studentID;
        this.name = name;
        this.department = department;
        this.marks = marks;
    }
    public int getStudentID() { return studentID; }
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getMarks() { return marks; }
}

// Controller: Handles all database operations
class StudentController {
    private static final String URL = "jdbc:mysql://localhost:3306/testdb";
    private static final String USER = "root";
    private static final String PASS = "password";

    public void addStudent(Student s) {
        String sql = "INSERT INTO Student VALUES (?, ?, ?, ?)";
        try (Connection con = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement ps = con.prepareStatement(sql)) {
            ps.setInt(1, s.getStudentID());
            ps.setString(2, s.getName());
            ps.setString(3, s.getDepartment());
            ps.setDouble(4, s.getMarks());
            ps.executeUpdate();
            System.out.println("Student Added Successfully!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void viewStudents() {
        String sql = "SELECT * FROM Student";
        try (Connection con = DriverManager.getConnection(URL, USER, PASS);
             Statement st = con.createStatement();
             ResultSet rs = st.executeQuery(sql)) {

            System.out.println("\nStudentID | Name | Department | Marks");
            while (rs.next()) {
                System.out.println(rs.getInt("StudentID") + " | " +
                        rs.getString("Name") + " | " +
                        rs.getString("Department") + " | " +
                        rs.getDouble("Marks"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void updateStudent(int id, double marks) {
        String sql = "UPDATE Student SET Marks=? WHERE StudentID=?";
        try (Connection con = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement ps = con.prepareStatement(sql)) {
            ps.setDouble(1, marks);
            ps.setInt(2, id);
            ps.executeUpdate();
            System.out.println("Student Marks Updated!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void deleteStudent(int id) {
        String sql = "DELETE FROM Student WHERE StudentID=?";
        try (Connection con = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement ps = con.prepareStatement(sql)) {
            ps.setInt(1, id);
            ps.executeUpdate();
            System.out.println("Student Deleted!");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

// View: User interface for interacting with students
class StudentView {
    private StudentController controller = new StudentController();
    private Scanner sc = new Scanner(System.in);

    public void showMenu() {
        boolean exit = false;
        while (!exit) {
            System.out.println("\n=== STUDENT MANAGEMENT MENU ===");
            System.out.println("1. Add Student");
            System.out.println("2. View Students");
            System.out.println("3. Update Student Marks");
            System.out.println("4. Delete Student");
            System.out.println("5. Exit");
            System.out.print("Enter choice: ");
            int ch = sc.nextInt();

            switch (ch) {
                case 1 -> {
                    System.out.print("Enter ID: ");
                    int id = sc.nextInt();
                    sc.nextLine();
                    System.out.print("Enter Name: ");
                    String name = sc.nextLine();
                    System.out.print("Enter Department: ");
                    String dept = sc.nextLine();
                    System.out.print("Enter Marks: ");
                    double marks = sc.nextDouble();
                    controller.addStudent(new Student(id, name, dept, marks));
                }
                case 2 -> controller.viewStudents();
                case 3 -> {
                    System.out.print("Enter Student ID to Update: ");
                    int id = sc.nextInt();
                    System.out.print("Enter New Marks: ");
                    double marks = sc.nextDouble();
                    controller.updateStudent(id, marks);
                }
                case 4 -> {
                    System.out.print("Enter Student ID to Delete: ");
                    int id = sc.nextInt();
                    controller.deleteStudent(id);
                }
                case 5 -> exit = true;
                default -> System.out.println("Invalid Choice!");
            }
        }
    }
}

// ========================= MAIN CLASS =========================
public class JDBCFullDemo {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("Choose Task:");
        System.out.println("1. Fetch Employee Data");
        System.out.println("2. Product CRUD Operations");
        System.out.println("3. Student Management (MVC)");
        System.out.print("Enter choice: ");
        int choice = sc.nextInt();

        switch (choice) {
            case 1 -> JDBCConnectionDemo.fetchEmployees();
            case 2 -> ProductCRUD.manageProducts();
            case 3 -> new StudentView().showMenu();
            default -> System.out.println("Invalid choice!");
        }
    }
}
