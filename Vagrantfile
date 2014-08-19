# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version "= 1.6.3"

#require_plugin is deprecated, but just note which plugins have to be installed
#Vagrant.require_plugin 'vagrant-omnibus'
#Vagrant.require_plugin 'vagrant-vbguest'
#Vagrant.require_plugin 'vagrant-hostsupdater'
#vagrant plugin install vagrant-berkshelf --plugin-version '>= 2.0.1'
#Vagrant.require_plugin 'vagrant-berkshelf'


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.omnibus.chef_version = "11.10.4"
  config.hostsupdater.remove_on_suspend = true
  config.berkshelf.enabled = true

  application_path="../php-sample-app"

  CHEF_NODES_PATH=File.dirname(__FILE__) + "/nodes/"
  Dir::glob("#{CHEF_NODES_PATH}*.json").each do |file|
    vagrant_json = JSON.parse(File.read(file))
    config.vm.define vagrant_json["name"] do |node|
      node.vm.hostname = vagrant_json["fqdn"]
      node.vm.box = vagrant_json["box"]
      node.vm.network :private_network, ip: vagrant_json["ipaddress"]
      node.vm.synced_folder "application_path",
        "/vagrant",
        create:true,
        owner:'www-data',
        group:'www-data',
        mount_options: ['dmode=755','fmode=644']
        node.vm.provision :chef_solo do |chef|
          chef.cookbooks_path = "./cookbooks"
          chef.data_bags_path = "./data_bags"
          chef.synced_folder_type="rsync"
          chef.environments_path="./environments"
          chef.roles_path = "./roles"
          chef.run_list = vagrant_json.delete('run_list')
          chef.environment=vagrant_json.delete('environment')
          #opsworks ssl support insteads
          if vagrant_json.has_key?("deploy") then
            vagrant_json["deploy"].each do |application_name,deploy_values|
            if deploy_values.has_key?("ssl_support") && deploy_values["ssl_support"] then
              if deploy_values.has_key?("domains") && !deploy_values["domains"].first.empty? then
                ssl_certificate_path=File.dirname(__FILE__)+"/files/default/"+deploy_values["domains"].first+"/ssl/server.crt"
                ssl_certificate_key_path=File.dirname(__FILE__)+"/files/default/"+deploy_values["domains"].first+"/ssl/server.key"
                ssl_certificate_ca_path=File.dirname(__FILE__)+"/files/default/"+deploy_values["domains"].first+"/ssl/server.ca"
                if File.exists?(ssl_certificate_path) && File.exists?(ssl_certificate_key_path) then
                  deploy_values["ssl_certificate"] = File.read(ssl_certificate_path)
                  deploy_values["ssl_certificate_key"] = File.read(ssl_certificate_key_path)
                  if File.exists?(ssl_certificate_ca_path)
                    deploy_values["ssl_certificate_ca"] = File.read(ssl_certificate_ca_path)
                  end
                else
                  raise Vagrant::Errors::VagrantError.new,
                   "ssl files not exist@"+ssl_certificate_path+":"+ssl_certificate_key_path+"\n"
                end
              else
               raise Vagrant::Errors::VagrantError.new,
                "deploy value ssl_support is true unless other options are enough!\n"
              end
            end
          end
          chef.json = vagrant_json
        end
      end
    end
  end
end
