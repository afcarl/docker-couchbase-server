# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_NAME = ENV['BOX_NAME'] || "precise64"
BOX_URI = ENV['BOX_URI'] || "ubuntu/precise64"
AWS_REGION = ENV['AWS_REGION'] || "us-east-1"
AWS_AMI    = ENV['AWS_AMI']    || "ami-d0f89fb9"
FORWARD_DOCKER_PORTS = ENV['FORWARD_DOCKER_PORTS']

Vagrant::Config.run do |config|
  # Setup virtual machine
  config.vm.box = BOX_NAME

  # Provision Docker and new kernel if deployment was not done.
  # It is assumed Vagrant can successfully launch the provider instance.
  if Dir.glob("#{File.dirname(__FILE__)}/.vagrant/machines/default/*/id").empty?

    # Add lxc-docker package
    pkg_cmd = "wget -q -O - https://get.docker.io/gpg | apt-key add -;" \
      "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list;" \
      "apt-get update -qq; apt-get install -q -y --force-yes lxc-docker; "

    # Add guest additions for VirtualBox provider
    if ENV["VAGRANT_DEFAULT_PROVIDER"].nil? && ARGV.none? { |arg| arg.downcase.start_with?("--provider") }

      # Install dependencies and set up Docker for Couchbase Server
      pkg_cmd << "apt-get install -q -y git vim; " \
        "sed -i.bak '/description     \"Docker daemon\"/ a\\limit nofile        262144 262144' /etc/init/docker.conf; " \
        "sed -i.bak '/description     \"Docker daemon\"/ a\\limit memlock       unlimited unlimited' /etc/init/docker.conf; " \
        "sed -i.back '/description     \"Docker daemon\"/ a\\# Following 2 lines added by docker-couchbase-server' /etc/init/docker.conf; "\
        "mkdir /home/couchbase-server; "\
        "chown 999:999 /home/couchbase-server;"

      pkg_cmd << "echo 'Downloading VBox Guest Additions...'; " \
        "wget -q http://dlc.sun.com.edgesuite.net/virtualbox/4.2.12/VBoxGuestAdditions_4.2.12.iso; "
      # Prepare the VM to add guest additions after reboot
      pkg_cmd << "echo -e 'mount -o loop,ro /home/vagrant/VBoxGuestAdditions_4.2.12.iso /mnt\n" \
        "echo yes | /mnt/VBoxLinuxAdditions.run\numount /mnt\n" \
          "rm /root/guest_additions.sh; ' > /root/guest_additions.sh; " \
        "chmod 700 /root/guest_additions.sh; " \
        "sed -i -E 's#^exit 0#[ -x /root/guest_additions.sh ] \\&\\& /root/guest_additions.sh#' /etc/rc.local; " \
        "echo 'Installation of VBox Guest Additions is proceeding in the background.'; " \
        "echo '\"vagrant reload\" can be used in about 2 minutes to activate the new guest additions.'; "
    end
    # Restart VM
    pkg_cmd << "shutdown -r +1; "
    config.vm.provision :shell, :inline => pkg_cmd
  end
end

# FIXME: change this to the new VAGRANT version requirement
Vagrant::VERSION >= "1.1.0" and Vagrant.configure("2") do |config|
  config.vm.provider :aws do |aws, override|
    aws.access_key_id = ENV["AWS_ACCESS_KEY_ID"]
    aws.secret_access_key = ENV["AWS_SECRET_ACCESS_KEY"]
    aws.keypair_name = ENV["AWS_KEYPAIR_NAME"]
    override.ssh.private_key_path = ENV["AWS_SSH_PRIVKEY"]
    override.ssh.username = "ubuntu"
    aws.region = AWS_REGION
    aws.ami    = AWS_AMI
    aws.instance_type = "t1.micro"
  end

  config.vm.provider :rackspace do |rs|
    config.ssh.private_key_path = ENV["RS_PRIVATE_KEY"]
    rs.username = ENV["RS_USERNAME"]
    rs.api_key  = ENV["RS_API_KEY"]
    rs.public_key_path = ENV["RS_PUBLIC_KEY"]
    rs.flavor   = /512MB/
    rs.image    = /Ubuntu/
  end

  config.vm.provider :vmware_fusion do |f, override|
    override.vm.box = BOX_NAME
    override.vm.synced_folder ".", "/vagrant", disabled: true
    f.vmx["displayName"] = "docker-couchbase-server"
  end

  config.vm.provider :virtualbox do |vb|
    config.vm.box = BOX_NAME
  end
end

if !FORWARD_DOCKER_PORTS.nil?
  Vagrant::VERSION < "1.1.0" and Vagrant::Config.run do |config|
    (49000..49900).each do |port|
      config.vm.forward_port port, port
    end
  end

  Vagrant::VERSION >= "1.1.0" and Vagrant.configure("2") do |config|
    (49000..49900).each do |port|
      config.vm.network :forwarded_port, :host => port, :guest => port
    end
  end
end
