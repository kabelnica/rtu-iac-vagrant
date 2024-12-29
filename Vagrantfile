Vagrant.configure("2") do |config|

    # Specify that we require a Ubuntu 22.04 VM
    config.vm.box = "ubuntu/jammy64"
  
    # Specify a static private IP (switch to DHCP if convenient)
    config.vm.network "private_network", ip: "192.168.33.103"
  
    # Specify machine parameters
    config.vm.provider "virtualbox" do |vb|
      vb.name = "IaC CRUD VM 161RDB025"
      vb.memory = "2048"
      vb.cpus = 2
    end
  
    # Provisioning block
    config.vm.provision "shell", inline: <<-SHELL
      # Update and upgrade packages
      sudo apt-get update -y
      sudo apt-get upgrade -y
  
      # Install Nginx
      sudo apt-get install -y nginx
  
      # Install MySQL
      sudo apt-get install -y mysql-server
      sudo mysql -e "CREATE DATABASE example_db;"
      sudo mysql -e "CREATE USER 'example_user'@'localhost' IDENTIFIED BY 'password';"
      sudo mysql -e "GRANT ALL PRIVILEGES ON example_db.* TO 'example_user'@'localhost';"
      sudo mysql -e "FLUSH PRIVILEGES;"
  
      # Install PHP and required extensions
      sudo apt-get install -y php-fpm php-mysql
  
      # Configure Nginx to use PHP
      sudo bash -c 'cat > /etc/nginx/sites-available/default' << 'EOF'
  server {
      listen 80;
      server_name localhost;
      root /var/www/html;
  
      index index.php index.html index.htm;
  
      location / {
          try_files $uri $uri/ =404;
      }
  
      location ~ \.php$ {
          include snippets/fastcgi-php.conf;
          fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          include fastcgi_params;
      }
  
      location ~ /\.ht {
          deny all;
      }
  }
EOF
  
      sudo systemctl restart nginx
  
# Create a basic PHP app
sudo mkdir -p /var/www/html
sudo bash -c 'cat > /var/www/html/index.php' << 'EOF'
<?php
$servername = "localhost";
$username = "example_user";
$password = "password";
$dbname = "example_db";

$conn = new mysqli($servername, $username, $password, $dbname);

if ($conn->connect_error) {
die("Connection failed: " . $conn->connect_error);
}

// Handle Create
if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST["name"])) {
$name = $_POST["name"];
$sql = "INSERT INTO example_table (name) VALUES ('$name')";
if ($conn->query($sql) === TRUE) {
    echo "New record created successfully";
} else {
    echo "Error: " . $sql . "<br>" . $conn->error;
}
}

// Handle Update
if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST["update_id"]) && isset($_POST["update_name"])) {
$id = $_POST["update_id"];
$name = $_POST["update_name"];
$sql = "UPDATE example_table SET name='$name' WHERE id=$id";
if ($conn->query($sql) === TRUE) {
    echo "Record updated successfully";
} else {
    echo "Error: " . $sql . "<br>" . $conn->error;
}
}

// Handle Delete
if ($_SERVER["REQUEST_METHOD"] == "POST" && isset($_POST["delete_id"])) {
$id = $_POST["delete_id"];
$sql = "DELETE FROM example_table WHERE id=$id";
if ($conn->query($sql) === TRUE) {
    echo "Record deleted successfully";
} else {
    echo "Error: " . $sql . "<br>" . $conn->error;
}
}

// Display records
$sql = "SELECT id, name FROM example_table";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
echo "<h1>Database Records</h1><ul>";
while($row = $result->fetch_assoc()) {
    echo "<li>ID: " . $row["id"]. " - Name: " . $row["name"] .
         " <form style='display:inline;' method='POST'>
             <input type='hidden' name='delete_id' value='" . $row["id"] . "'>
             <button type='submit'>Delete</button>
           </form>
           <form style='display:inline;' method='POST'>
             <input type='hidden' name='update_id' value='" . $row["id"] . "'>
             <input type='text' name='update_name' placeholder='New Name' required>
             <button type='submit'>Update</button>
           </form>
         </li>";
}
echo "</ul>";
} else {
echo "0 results";
}

$conn->close();
?>
<form method="POST">
<input type="text" name="name" placeholder="Enter Name" required>
<button type="submit">Add</button>
</form>
EOF
  
    # Create example database table
    sudo mysql -u root -e "USE example_db; CREATE TABLE example_table (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL);"
SHELL
  
end
  