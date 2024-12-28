Vagrant.configure("2") do |config|

  # Specify that we require a Ubuntu 22.04 VM
  config.vm.box = "ubuntu/jammy64"

  # Specify a static private IP (switch to DCHP if convenient)
  config.vm.network "private_network", ip: "192.168.33.103"

  # Specify machine parameters as specified in the task
  config.vm.provider "virtualbox" do |vb|
    vb.name = "IaC CRUD VM 161RDB025"
    vb.memory = "2048"
    vb.cpus = 2
  end

  # Provisioning: Install Nginx, MySQL, PHP, and setup a simple CRUD app
  config.vm.provision "shell", inline: <<-SHELL
    # Update packages
    sudo apt-get update

    # Install Nginx
    sudo apt-get install -y nginx

    # Install MySQL
    sudo apt-get install -y mysql-server
    sudo mysql_secure_installation <<EOF

    y
    rootpassword
    rootpassword
    y
    y
    y
    y
    EOF

    # Create a database and user for the CRUD app
    sudo mysql -u root -prootpassword -e "CREATE DATABASE crud_app;"
    sudo mysql -u root -prootpassword -e "CREATE USER 'crud_user'@'localhost' IDENTIFIED BY 'password';"
    sudo mysql -u root -prootpassword -e "GRANT ALL PRIVILEGES ON crud_app.* TO 'crud_user'@'localhost';"
    sudo mysql -u root -prootpassword -e "FLUSH PRIVILEGES;"

    # Install PHP and necessary modules
    sudo apt-get install -y php-fpm php-mysql

    # Configure Nginx to use PHP
    sudo tee /etc/nginx/sites-available/default > /dev/null <<NGINX_CONF
    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.php index.html index.htm;

        server_name _;

        location / {
            try_files \$uri \$uri/ =404;
        }

        location ~ \.php\$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.ht {
            deny all;
        }
    }
    NGINX_CONF

    # Restart Nginx
    sudo systemctl restart nginx

    # Set up a basic PHP app for CRUD functionality
    sudo mkdir -p /var/www/html
    sudo tee /var/www/html/index.php > /dev/null <<'PHP_APP'
    <?php
    $conn = new mysqli("localhost", "crud_user", "password", "crud_app");

    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    $action = $_GET['action'] ?? '';
    if ($action == "create") {
        $conn->query("CREATE TABLE IF NOT EXISTS items (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255))");
        echo "Table created successfully!";
    } elseif ($action == "insert") {
        $conn->query("INSERT INTO items (name) VALUES ('Sample Item')");
        echo "Item inserted successfully!";
    } elseif ($action == "read") {
        $result = $conn->query("SELECT * FROM items");
        while ($row = $result->fetch_assoc()) {
            echo $row['id'] . ": " . $row['name'] . "<br>";
        }
    } elseif ($action == "update") {
        $conn->query("UPDATE items SET name='Updated Item' WHERE id=1");
        echo "Item updated successfully!";
    } elseif ($action == "delete") {
        $conn->query("DELETE FROM items WHERE id=1");
        echo "Item deleted successfully!";
    } else {
        echo "Welcome to the CRUD app!";
    }
    ?>
    PHP_APP

    # Adjust permissions
    sudo chown -R www-data:www-data /var/www/html

  SHELL

end
