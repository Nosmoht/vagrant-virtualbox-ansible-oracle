# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

domain = "example.com"

boxes = [
	{ :name => 'ora11gr2oel6', :box => 'ntbc-oel65', :ip => '10.0.0.100', :cpus => 1, :memory =>  1024 },
	{ :name => 'ora11gr2oel7', :box => 'ntbc-oel7', :ip => '10.0.0.101', :cpus => 1, :memory =>  1024 },
	{ :name => 'ora12cr1oel6', :box => 'ntbc-oel65', :ip => '10.0.0.102', :cpus => 1, :memory =>  1024 },
	{ :name => 'ora12cr1oel7', :box => 'ntbc-oel7', :ip => '10.0.0.103', :cpus => 1, :memory =>  1024 },		
]

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	unless Vagrant.has_plugin?("vagrant-hostmanager")
		raise 'Plugin not found. Run: vagrant plugin install vagrant-hostmanager'
	end
	config.hostmanager.enabled = true
	config.hostmanager.manage_host = true
	config.hostmanager.ignore_private_ip = false

	if Vagrant.has_plugin?("vagrant-cachier")
		config.cache.scope = :box
		config.cache.enable :yum
	else
		puts "[-] WARN: This would be much faster if you ran 'vagrant plugin install vagrant-cachier' first"
	end

	boxes.each do |opts|
		config.vm.define opts[:name] do |config|
			hostname = "#{opts[:name]}.#{domain}"
			config.vm.box = "#{opts[:box]}"
			config.vm.hostname = "#{hostname}"
			config.hostmanager.aliases = "#{opts[:name]}"
			config.vm.network :private_network, ip: "#{opts[:ip]}"
			config.vm.provider "virtualbox" do |vb|
				vb.name = opts[:name]
				vb.customize ["modifyvm", :id, "--ioapic", "on"]
			        vb.customize ["modifyvm", :id, "--memory", opts[:memory]] if opts[:memory]
		        	vb.customize ["modifyvm", :id, "--cpus", opts[:cpus]] if opts[:cpus]
			end
			config.vm.synced_folder "/home/ntbc/oracle", "/share", type: "nfs"
			#Fix for Ansible bug resulting in an encoding error
			ENV['PYTHONIOENCODING'] = "utf-8"
			config.vm.provision "ansible" do |ansible|
				ansible.limit = "#{opts[:name]}"
				ansible.playbook = "ansible/oracle-server.yml"
				ansible.inventory_path = "ansible/hosts"
			end
	    end
	end
end
