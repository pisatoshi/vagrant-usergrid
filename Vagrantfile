# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define :usergrid do |usergrid|
    usergrid.vm.box = "centos"
    usergrid.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box"
    usergrid.vm.provider :virtualbox do |vb|
      vb.gui = true
      usergrid.vm.network :private_network, ip: "192.168.10.100"
      usergrid.vm.network :forwarded_port, guest: 8080, host: 18080, host_ip: "127.0.0.1", id: "usergrid", auto_correct: true
      vb.customize ["modifyvm", :id, "--memory", "512"]
    end
  end

  config.vm.define :usergrid_aws do |usergrid_aws|
    ssh_username = "root"
    aws_access_key = ENV['AWS_ACCESS_KEY']
    aws_secret_key = ENV['AWS_SECRET_KEY']
    aws_keypair_name = ENV['AWS_KEYPAIR_NAME']
    private_key_path = ENV['AWS_PRIVATE_KEY_PATH']
    usergrid_aws.vm.box = "aws"
    usergrid_aws.vm.provider :aws do |aws, override|
      aws.tags = { 
        'Name' => 'usergrid'
      }
      aws.region = 'ap-northeast-1'
      aws.instance_type = 'm1.small'
      aws.ami = 'ami-05197104'
      aws.security_groups = ['usergrid']

      aws.access_key_id = aws_access_key
      aws.secret_access_key = aws_secret_key
      aws.keypair_name = aws_keypair_name
      override.ssh.username = ssh_username
      override.ssh.private_key_path = private_key_path
    end
  end

  config.vm.provision :shell, :inline => <<-EOT
    rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    yum -y upgrade

    yum -y install wget
    #yum -y install java-1.7.0-openjdk
    #yum remove java-1.7.0-openjdk

    if [ ! -e /usr/java/jdk1.7.0_51 ]; then
      wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/7u51-b13/jdk-7u51-linux-x64.rpm" -O /opt/jdk-7-linux-x64.rpm
      rpm -Uvh /opt/jdk-7-linux-x64.rpm
      alternatives --install /usr/bin/java java /usr/java/jdk1.7.0_51/jre/bin/java 20000
      alternatives --install /usr/bin/jar jar /usr/java/jdk1.7.0_51/bin/jar 20000
      alternatives --install /usr/bin/javac javac /usr/java/jdk1.7.0_51/bin/javac 20000
      alternatives --install /usr/bin/javaws javaws /usr/java/jdk1.7.0_51/jre/bin/javaws 20000
      alternatives --set java /usr/java/jdk1.7.0_51/jre/bin/java
      alternatives --set javaws /usr/java/jdk1.7.0_51/jre/bin/javaws
      alternatives --set javac /usr/java/jdk1.7.0_51/bin/javac
      alternatives --set jar /usr/java/jdk1.7.0_51/bin/jar
    fi

    if [ ! -e /usr/local/apache-maven-3.2.1-bin.tar.gz ]; then
      wget 'http://ftp.tsukuba.wide.ad.jp/software/apache/maven/maven-3/3.2.1/binaries/apache-maven-3.2.1-bin.tar.gz' -O /usr/local/apache-maven-3.2.1-bin.tar.gz
      tar xvfz /usr/local/apache-maven-3.2.1-bin.tar.gz -C /usr/local

      wget 'http://ftp.yz.yamagata-u.ac.jp/pub/network/apache/maven/maven-3/3.0.5/binaries/apache-maven-3.0.5-bin.tar.gz' -O /usr/local/apache-maven-3.0.5-bin.tar.gz
      tar xvfz /usr/local/apache-maven-3.0.5-bin.tar.gz -C /usr/local

    fi

    rm -f /etc/profile.d/maven.sh
    echo 'export JAVA_HOME=/usr/java/jdk1.7.0_51' >> /etc/profile.d/maven.sh
    echo 'export M2_HOME=/usr/local/apache-maven-3.0.5' >> /etc/profile.d/maven.sh
    echo 'export M2=$M2_HOME/bin' >> /etc/profile.d/maven.sh
    echo 'export PATH=$M2:$PATH' >> /etc/profile.d/maven.sh

    wget 'https://github.com/usergrid/usergrid/archive/master.zip' -O /usr/local/usergrid.zip
    cd /usr/local/
    unzip usergrid.zip
    cd usergrid-master/stack
    mvn clean install -DskipTests=true
  EOT
end
