# General project settings
#################################

# NOTE: Remember to run "vagrant reload" after changing any of these

# Host Only Network IP Address
ip_address = "10.1.1.1"
domain = ".local"

# The project name is base for directories, hostname and alike
project_name = "sampleproject"

# MySQL and PostgreSQL password 
database_password = "password"

# Vagrant configuration
#################################
Vagrant.configure("2") do |config|
    # Enable Berkshelf support
    # http://berkshelf.com/
    config.berkshelf.enabled = true

    # Define VM box to use
    config.vm.box = "ubuntu-trusty"
    config.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"

    # Set share folder
    config.vm.synced_folder "../" + project_name, "/var/www/" + project_name + "/", :mount_options => ["dmode=777", "fmode=666"] 

    # Use hostonly network with a static IP Address and enable
    # hostmanager so we can have a custom domain for the server
    # by modifying the host machines hosts file
    # https://github.com/smdahlen/vagrant-hostmanager
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.vm.define project_name do |node|
        # local domain
        node.vm.hostname = project_name + domain
        # ip address
        node.vm.network :private_network, ip: ip_address
        #apache
        node.vm.network :forwarded_port, guest: 80, host: 8080,
          auto_correct: true
        # rabbitmq administration panel
        node.vm.network :forwarded_port, guest: 15672, host: 15672,
          auto_correct: true
        # elasticsearch
        node.vm.network :forwarded_port, guest: 9200, host: 9200,
          auto_correct: true
        # mongodb
        node.vm.network :forwarded_port, guest: 27017, host: 27017,
          auto_correct: true

        node.hostmanager.aliases = [ "www." + project_name + domain ]
    end

    config.vm.provider "virtualbox" do |vb|
      vb.name = project_name
      vb.memory = 2048
      vb.cpus = 2
      vb.gui = false
    end

    config.vm.provision :hostmanager

    # Make sure that the newest version of Chef have been installed
    config.vm.provision :shell, :inline => "apt-get update -qq --no-upgrade --yes"
    config.vm.provision :shell, :inline => "gem install chef"

    # Povision using Chef Solo
    config.vm.provision :chef_solo do |chef|
        chef.add_recipe "ntp"
        chef.add_recipe "apache2"
        chef.add_recipe "apache2::mod_ssl"
        chef.add_recipe "apache2::mod_rewrite"
        chef.add_recipe "apache2::mod_headers"
        chef.add_recipe "apache2::mod_deflate"
        chef.add_recipe "php"
        chef.add_recipe "memcached"
        chef.add_recipe "postfix"
        chef.add_recipe "phpunit"
        chef.add_recipe "mysql::client"
        chef.add_recipe "mysql::server"
        chef.add_recipe "mongodb"
        chef.add_recipe "java"
        chef.add_recipe "elasticsearch"
        chef.add_recipe "varnish"
        chef.add_recipe "rabbitmq"
        chef.add_recipe "users"
        chef.add_recipe "logrotate"

        chef.json = {
            :ntp => {
                :servers => ['0.ubuntu.pool.ntp.org', '1.ubuntu.pool.ntp.org', '2.ubuntu.pool.ntp.org', '3.ubuntu.pool.ntp.org']
            },

            :server => {
                :name           	=> project_name,

                ##### Apache VHost #####
                # Server name and alias
                :server_name    	=> project_name + domain,
                :server_aliases 	=> [ "www." + project_name + domain ],

                # DocRoot
                :docroot        	=> "/var/www/" + project_name + "/web",

                ##### Packages needed #####
                 # Ubuntu
                :apt_pkgs       	=> %w{ vim git screen curl wget mc telnet },

                # Node/NPM
                :npm_pkgs   		=> %w{ grunt-cli bower },

                #Needs to match mysql password for phpMyAdmin install
                :db_password 		=> database_password
            },

            :apache => {
                :docroot_dir             => "/var/www/" + project_name + "/web",
                :listen_ports            => ["80", "443"],
                :contact                 => "webmaster@" + project_name + "",
                :default_modules         => %w{ status alias auth_basic authn_file autoindex dir env mime negotiation setenvif rewrite ssl },
                :web_app => {
                    :name => "ads",
                    :server_name => project_name + ".local",
                    :server_aliases => "www." + project_name + ".local",
                    :docroot => "/var/www/" + project_name + "/web",
                    
                    :enable => "true"
                }
            },

            :xdebug => {
                :cli_color               => 1,
                :scream                  => 0,
                :remote_enable           => "On",
                :remote_autostart        => 0,
                :remote_mode             => "req",
                :remote_connect_back     => 1,
                :idekey                  => "vagrant",
                :file_link_format        => "txmt://open?url=file://%f&line=%1",
                :profiler_enable_trigger => 0,
                :profiler_enable         => 0,
                :profiler_output_dir     => "/tmp/cachegrind"
            },

            # https://github.com/opscode-cookbooks/php/blob/master/attributes/default.rb
            :php => {
                :url                     => "http://us1.php.net/get",
                :version                 => "5.6.2",
                :checksum                => "f0b54371fae6d4b6b99fb3d360f086ac",
                # PHP modules
                :packages                => %w{ php5 php5-dev php5-cli php-pear php5-apcu php5-mysql php5-curl php5-mcrypt php5-memcached php5-gd php5-json php5-mongo },

                # Apache 2.4's conf directory for modules
                :ext_conf_dir            => "/etc/php5/mods-available",
                :directives              => {
                    "date.timezone" => "Europe/Warsaw",
                }
            },

            :phpunit => {
                :install_method          => 'composer'
            },

            # mysql database
            :mysql => {
                :server_root_password    => database_password,
                :server_repl_password    => database_password,
                :server_debian_password  => database_password,
                :bind_address            => ip_address,
                :allow_remote_root       => true
            },

            :java => {
                :install_flavor          => "oracle",
                :jdk_version             => "8",
                :oracle                  => {
                    :accept_oracle_download_terms => true
                }
            },

            :elasticsearch => {
                :cluster => { :name => "elastic-vagrant" },

                :version => "1.3.4",

                :plugins => {
                    'karmi/elasticsearch-paramedic' => {
                      :url => 'https://github.com/karmi/elasticsearch-paramedic/archive/master.zip'
                    }
                },

                :limits => {
                    :nofile  => 1024,
                    :memlock => 512
                },

                :logging => {
                    :discovery => 'TRACE',
                    'index.indexing.slowlog' => 'INFO, index_indexing_slow_log_file'
                },
            },

            :mongodb => {
                :package_version => "2.6.4",
                :package_name => "mongodb-org"
            }
        }
    end
end
