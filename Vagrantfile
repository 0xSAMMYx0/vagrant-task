Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu:22.04"
  
  config.vm.provider "docker" do |d|
    d.image = "ubuntu:22.04"
    d.create_args = ["--platform", "linux/arm64"]
    d.has_ssh = false
    d.entry_point = "bash"
    d.cmd = ["-c", "apt-get update -y && apt-get install -y openjdk-17-jdk && cd /vagrant && chmod +x gradlew && ./gradlew bootRun || tail -f /dev/null"]
  end

  config.vm.network "forwarded_port", guest: 8080, host: 8080
end
