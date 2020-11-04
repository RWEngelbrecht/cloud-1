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
    2. Connect bucket to instance:
        1. Create bucket on GCP console.
        2. Create service account and download keyfile related to account.
        3. SSH into WP instance.
        - Note: to change the following files, you need to be logged in as super user.
        4. Add the following to `wp-config.php`:
           ```
           define( 'AS3CF_SETTINGS', serialize( array('provider' => 
            'gcp', 'key-file-path' => '/etc/file.json',) ) );
           ```
