# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'erb'
require 'ostruct'

def render_template(template_path, variables)
  template = File.read(File.expand_path(template_path, __dir__))
  namespace = OpenStruct.new(variables)
  ERB.new(template, trim_mode: '-').result(namespace.instance_eval { binding })
end

#Define the list of machines
slurm_cluster = {
    :controller => {
        :type => "controller",
        :hostname => "controller",
        :ipaddress => "10.10.10.10"
    },
    :worker1 => {
        :type => "worker",
        :hostname => "worker1",
        :ipaddress => "10.10.10.101"
    },
    :worker2 => {
        :type => "worker",
        :hostname => "worker2",
        :ipaddress => "10.10.10.102"
    }
}

mariadb_pass = ENV['SLURM_MARIADB_PASS']

# Generate hosts file entries from slurm_cluster
$hosts_setup = ""
slurm_cluster.each do |key, machine|
  $hosts_setup += "echo \"#{machine[:ipaddress]}    #{machine[:hostname]}\" >> /etc/hosts\n"
end

# Base Linux configuration
$base_linux_content = render_template("templates/all_nodes_linux.sh.erb", {
  hosts_setup: $hosts_setup
})


$munge_controller_script = render_template("templates/controller_munge_install.sh.erb", {})
$munge_worker_script = render_template("templates/worker_munge_install.sh.erb", {})


# Slurm configuration file for all nodes
$slurm_config_file_content = render_template("templates/all_nodes_slurm.conf.erb", {
  controller_hostname: slurm_cluster[:controller][:hostname],
  cluster: slurm_cluster
})

# MariaDB installation and Slurmdbd configuration
$controller_mariadb_script = render_template("templates/controller_mariadb.sh.erb", {
  mariadb_pass: mariadb_pass
})

$slurmdbd_config_file_content = render_template("templates/controller_slurmdbd.conf.erb", {
  mariadb_pass: mariadb_pass
})

$slurmdbd_script = render_template("templates/controller_slurmdb.sh.erb", {
  mariadb_pass: mariadb_pass,
  slurmdbd_config_file_content: $slurmdbd_config_file_content
})

$slurmctld_script = render_template("templates/controller_slurmctl.sh.erb", {
  slurm_config_file_content: $slurm_config_file_content
})

$worker_slurmd_script = render_template("templates/worker_slurm_install.sh.erb", {
    slurm_config_file_content: $slurm_config_file_content
})


Vagrant.configure("2") do |global_config|
    global_config.vm.provider "virtualbox"
    slurm_cluster.each_pair do |name, options|
        global_config.vm.define name do |config|
            #VM configurations
            config.vm.box = "cloud-image/ubuntu-24.04"
            config.vm.hostname = "#{name}"
            config.vm.network :private_network, ip: options[:ipaddress]

            #VM specifications
            config.vm.provider :virtualbox do |v|
                v.cpus = 1
                v.memory = 2048
            end

            #VM provisioning
            config.vm.provision :shell, inline: $base_linux_content

            if options[:type] == "controller"
                config.vm.provision :shell, inline: $munge_controller_script
                config.vm.provision :shell, inline: $controller_mariadb_script
                config.vm.provision :shell, inline: $slurmdbd_script
                config.vm.provision :shell, inline: $slurmctld_script
            elsif options[:type] == "worker"
                config.vm.provision :shell, inline: $munge_worker_script
                config.vm.provision :shell, inline: $worker_slurmd_script
            end
        end
    end
end
