Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true 
  config.hostmanager.manage_host = true
  
### Jenkins UI VM ###
  config.vm.define "jenkins-ui" do |jenkinsui|
    jenkinsui.vm.box = "ubuntu/focal64"
    jenkinsui.vm.hostname = "jenkins-ui"
	jenkinsui.vm.network "private_network", ip: "192.168.56.11"
  end
  
### Jenkins Agent VM ###
  config.vm.define "jenkinsagent" do |jenkinsagent|
    jenkinsagent.vm.box = "ubuntu/focal64"
    jenkinsagent.vm.hostname = "jenkinsagent"
	jenkinsagent.vm.network "private_network", ip: "192.168.56.12"
  end

### SonarQube VM ###
config.vm.define "sonarqube" do |sonarqube|
  sonarqube.vm.box = "ubuntu/focal64"
  sonarqube.vm.hostname = "sonarqube"
  sonarqube.vm.network "private_network", ip: "192.168.56.13"
  sonarqube.vm.provider "virtualbox" do |sq|
     sq.memory = "3096"
     sq.cpus = 4
	 end
end 

end
