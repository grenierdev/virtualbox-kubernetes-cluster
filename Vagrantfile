prefix = "192.168.56."

masters = [
  ["master01", 10],
]

workers = [
  ["worker01", 20],
  ["worker02", 21],
  ["worker03", 22],
]

nfs = [
  ["nfs01", 30],
]


hostfile_script = [].concat(masters, workers, nfs).collect { |data| "echo " + prefix + data[1].to_s + " " + data[0] + ".grenier.local " + data[0] + " >> /etc/hosts" }.join("\n")

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.provision "shell", privileged: true, inline: hostfile_script
  
  masters.each do |data|
    config.vm.define name="#{data[0]}.grenier.local" do |h|
      h.vm.hostname = name
      #h.vm.disk :disk, size: "10GB", primary: true
      h.vm.network "public_network", ip: "#{prefix}#{data[1]}", hostname: true
      h.vm.provider "virtualbox" do |vb|
        vb.name = name
        vb.cpus = "4"
        vb.memory = "4096"
      end
    end
  end

  workers.each do |data|
    config.vm.define name="#{data[0]}.grenier.local" do |h|
      h.vm.hostname = name
      #h.vm.disk :disk, size: "25GB", primary: true
      h.vm.network "public_network", ip: "#{prefix}#{data[1]}", hostname: true
      h.vm.provider "virtualbox" do |vb|
        vb.name = name
        vb.cpus = "4"
        vb.memory = "2048"
      end
    end
  end

  nfs.each do |data|
    config.vm.define name="#{data[0]}.grenier.local" do |h|
      h.vm.hostname = name
      h.vm.disk :disk, size: "100GB", primary: true
      h.vm.network "public_network", ip: "#{prefix}#{data[1]}", hostname: true
      h.vm.provider "virtualbox" do |vb|
        vb.name = name
        vb.cpus = "4"
        vb.memory = "2048"
      end
    end
  end
end
