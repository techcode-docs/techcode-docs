To implement CRUD (Create, Read, Update, Delete) operations in PHP with a UI on a local server, follow the step-by-step guide below. We'll be using MySQLi for database interactions, without using PDO. This guide includes the setup for a local server, directory structure, file creation, and CRUD implementation.

# Step 1: Local Server Installation (XAMPP)

1. Download XAMPP: Download and install XAMPP from [https://www.apachefriends.org/download.html](https://www.apachefriends.org/download.html). This will set up Apache, PHP, and MySQL on your local machine.
   
2. Start XAMPP:
   - Open XAMPP Control Panel.
   - Start Apache (for PHP) and MySQL (for database).

3. Access Localhost: Open your browser and go to `http://localhost/` to ensure XAMPP is running properly.

4. Set up the database:
   - Go to `http://localhost/phpmyadmin`.
   - Create a new database called `crud_db`.
   - Run the following SQL query to create a `users` table:

   sql
   CREATE DATABASE crud_db;
   USE crud_db;
   
   CREATE TABLE users (
       id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
       name VARCHAR(100) NOT NULL,
       email VARCHAR(100) NOT NULL,
       created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
   );
   

# Step 2: Project Directory Structure

Now, we’ll create a directory structure to organize the PHP project effectively.

## Directory Structure:


/htdocs (this is in XAMPP directory)
  └── /crud_php
       ├── /config
       │   └── db.php
       ├── /public
       │   ├── index.php
       │   ├── create.php
       │   ├── read.php
       │   ├── update.php
       │   └── delete.php
       ├── /templates
       │   ├── header.php
       │   └── footer.php
       └── /css
           └── styles.css


# Step 3: File Creation and Coding

## 1. `/config/db.php`: Database Connection
This file manages the connection to the MySQL database.

php
<?php
// Database configuration
$host = "localhost";
$user = "root";
$pass = "";
$dbname = "crud_db";

// Create connection
$conn = new mysqli($host, $user, $pass, $dbname);

// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
?>


## 2. `/public/index.php`: Main File (Redirects to Read Page)
This is the entry point, typically used for redirection to the read page.

php
<?php
header('Location: read.php');
exit;


## 3. `/public/read.php`: Read Operation
This file displays a list of users from the `users` table.

php
<?php
include '../config/db.php';
include '../templates/header.php';

// Fetch all users
$sql = "SELECT * FROM users";
$result = $conn->query($sql);
?>

<h2>Users List</h2>
<a href="create.php">Add New User</a>
<table border="1" cellpadding="10" cellspacing="0">
    <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Email</th>
        <th>Created At</th>
        <th>Actions</th>
    </tr>
    <?php while($row = $result->fetch_assoc()): ?>
    <tr>
        <td><?= $row['id'] ?></td>
        <td><?= $row['name'] ?></td>
        <td><?= $row['email'] ?></td>
        <td><?= $row['created_at'] ?></td>
        <td>
            <a href="update.php?id=<?= $row['id'] ?>">Edit</a>
            <a href="delete.php?id=<?= $row['id'] ?>" onclick="return confirm('Are you sure?')">Delete</a>
        </td>
    </tr>
    <?php endwhile; ?>
</table>

<?php include '../templates/footer.php'; ?>


## 4. `/public/create.php`: Create Operation
This file allows the user to add new entries to the database.

php
<?php
include '../config/db.php';
include '../templates/header.php';

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $name = $_POST['name'];
    $email = $_POST['email'];

    if (!empty($name) && !empty($email)) {
        $sql = "INSERT INTO users (name, email) VALUES ('$name', '$email')";
        if ($conn->query($sql)) {
            header('Location: read.php');
        } else {
            echo "Error: " . $conn->error;
        }
    } else {
        echo "Please fill out all fields.";
    }
}
?>

<h2>Add New User</h2>
<form method="POST" action="">
    Name: <input type="text" name="name"><br>
    Email: <input type="text" name="email"><br>
    <input type="submit" value="Submit">
</form>

<?php include '../templates/footer.php'; ?>


## 5. `/public/update.php`: Update Operation
This file allows editing existing user data.

php
<?php
include '../config/db.php';
include '../templates/header.php';

$id = $_GET['id'];
$sql = "SELECT * FROM users WHERE id=$id";
$result = $conn->query($sql);
$row = $result->fetch_assoc();

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $name = $_POST['name'];
    $email = $_POST['email'];

    if (!empty($name) && !empty($email)) {
        $sql = "UPDATE users SET name='$name', email='$email' WHERE id=$id";
        if ($conn->query($sql)) {
            header('Location: read.php');
        } else {
            echo "Error: " . $conn->error;
        }
    } else {
        echo "Please fill out all fields.";
    }
}
?>

<h2>Update User</h2>
<form method="POST" action="">
    Name: <input type="text" name="name" value="<?= $row['name'] ?>"><br>
    Email: <input type="text" name="email" value="<?= $row['email'] ?>"><br>
    <input type="submit" value="Update">
</form>

<?php include '../templates/footer.php'; ?>


## 6. `/public/delete.php`: Delete Operation
This file handles the deletion of user data.

php
<?php
include '../config/db.php';

$id = $_GET['id'];

$sql = "DELETE FROM users WHERE id=$id";
if ($conn->query($sql)) {
    header('Location: read.php');
} else {
    echo "Error deleting record: " . $conn->error;
}
?>


## 7. `/templates/header.php`: Header Template
This file contains the header and navigation that will be included on every page.

php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="../css/styles.css">
    <title>CRUD PHP Project</title>
</head>
<body>
    <header>
        <h1>CRUD PHP Application</h1>
        <nav>
            <a href="read.php">Home</a>
            <a href="create.php">Add User</a>
        </nav>
    </header>
    <main>


## 8. `/templates/footer.php`: Footer Template
This file contains the footer for the pages.

php
    </main>
    <footer>
        <p>&copy; <?= date("Y") ?> CRUD PHP Project</p>
    </footer>
</body>
</html>


## 9. `/css/styles.css`: Basic Styling
This file contains basic styling for the layout.

css
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
}

header {
    background-color: #333;
    color: white;
    padding: 15px;
    text-align: center;
}

nav a {
    color: white;
    margin: 0 15px;
    text-decoration: none;
}

main {
    padding: 20px;
}

table {
    width: 100%;
    border-collapse: collapse;
}

table, th, td {
    border: 1px solid black;
}

th, td {
    padding: 10px;
    text-align: left;
}

footer {
    background-color: #333;
    color: white;
    text-align: center;
    padding: 10px 0;
    position: fixed;
    width: 100%;
    bottom: 0;
}


# Step 4: Accessing the Application

1. Place the project folder `crud_php` inside the `htdocs` folder in your XAMPP directory.
   
2. Open your browser and go to: `http://localhost/crud_php/public/read.php` to view the list of users.

# Final Thoughts

- Additions: You can expand on this structure by adding user authentication, better validation, or AJAX for a more dynamic

 experience.
- Testing: Test the functionality by adding, updating, and deleting records from the interface.

This is a basic but complete PHP CRUD application without PDO, using MySQLi instead.