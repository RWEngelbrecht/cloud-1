# cloud-1
My first simple cloud infrastructure build with Google Cloud Platform

## Requirements:
- At least 2 running instances of website at all times.
- Evenly distribute traffic between active instances.
- Scale instances up or down depending on traffic.
- Logged in users will stay identified for the length of a normal session.
- CDN used to distribute static content.

## Setup with Google Cloud Platform:
1.  WordPress VM instance:
    1. Instantiate VM instance from Marketplace.
    2. Connect bucket to instance for media uploads:
        1. Create bucket on GCP console.
        2. Create service account and download keyfile related to account.
        3. SSH into WP instance.
        > Note: to change the following files, you need to be logged in as super user.
        4. Add the following to `wp-config.php`:
           ```
           define( 'AS3CF_SETTINGS', serialize( array('provider' => 
            'gcp', 'key-file-path' => '/etc/file.json',) ) );
           ```
        5. Create file according to 'key-file-path' and copy downloaded keyfile text into it.
        6. In a browser, navigate to external_ip_of_wordpress_instance/wp-admin, log in.
        7. Install and activate WP Offload Media Lite plugin.
        8. In the plugin settings, select the bucket you created.
    3. Connect SQL database to instance:
        1. Create SQL instance on GCP console, enabling private IP.
        2. Create new database for the instance, setting charset to 'utf8mb4' and collate to 'utfmb48_general_ci'.
        3. SSH into WP instance and add to `wp-config.php`:
            ```
            define( 'DB_NAME', '<SQL Database name>');
            define( 'DB_USER', '<SQL Database username>');
            define( 'DB_PASSWORD', '<SQL Database password>');
            define( 'DB_HOST', '<Private IP of SQL instance>');
            define( 'DB_CHARSET', 'utf8mb4');
            define( 'DB_COLLATE', 'utfmb48_general_ci');
            ```
    4. Install W3 Total Cache plugin:
        - Follow instructions regarding the `.htaccess` file.
        - Enable CDN.
    5. Forward HTTP traffic to HTTPS:
        1. Add the following to `wp-config.php`:
            ```
            define('FORCE_SSL_ADMIN', true);
                if (strpos($_SERVER['HTTP_X_FORWARDED_PROTO'], 'https') !== false)
                    $_SERVER['HTTPS']='on';
            ```
        2. Add to `.htaccess` file:
            ```
            RewriteEngine On
            RewriteCond %{HTTP:X-Forwarded-Proto}=http
            RewriteRule .* https://%{HTTP:Host}%{REQUEST_URI} [L,R=permanent]
            ```
2. Create image from WordPress instance:
    1. STOP WordPress VM instance created in previous step.
    2. From Compute Engine > Storage > Images in GCP console, create image.
    3. Select created WP VM instance as source disk.
3. Create instance template from image:
    1. From Compute Engine > Virtual machines > Instance templates, create instance template.
    2. Select custom image as boot disk, selecting created image.
    3. Allow both HTTP and HTTPS traffic.
4. Create instance group from instance template:
    1. From Compute Engine > Instance groups > Instance groups, create instance group.
    2. Set instance template as template created in previous step.
    3. Turn on Autoscaling.
    4. Set minimum and maximum number of instances as required.
    5. Create new health check for group.
5. Create firewall rule for external frontend access:
    1. From VPC Network > Firewall, create firewall rule.
    2. Set Direction to 'Ingress'.
    3. Allow Action on Match.
    4. Target all instances.
    5. Set Source Filter to IP Range.
    6. Set the range to `130.211.0.0./22`, `35.191.0.0/16`.
    7. Check TCP and set to port 80.
6. Set up GCP managed SSL certificate:
    1. From Network services > Load balancing, follow the 'advanced menu' link.
    2. From Certificates tab, create ssl certificate.
    3. Check 'Google-managed certificate'.
    4. Add appropriate domain name/IP.
7. Create HTTPS and HTTP Load balancer, making sure to redirect HTTP traffic to HTTPS, enable Cloud CDN and adding the SSL certificate created in the previous step.

## Diagram
![GCP Infrastructure Diagram](https://github.com/RWEngelbrecht/cloud-1/blob/main/cloud1_gcp_diagram.png)
