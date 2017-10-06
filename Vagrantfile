# version 2
Vagrant.configure(2) do |config|
  config.vm.box = 'ubuntu/xenial64'

  # 放入自己的 private key
  config.vm.provision 'shell' do |s|
    hadoop_ssh_pem_key = File.read('roles/common/templates/id_rsa')
    hadoop_ssh_pub_key = File.readlines('roles/common/templates/id_rsa.pub').first.strip

    if File.exist?("#{Dir.home}/.ssh/id_rsa.pub")
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    end
    shell_script = ''
    unless ssh_pub_key.nil?
      shell_script = <<-SHELL
        grep -q '#{ssh_pub_key}' .ssh/authorized_keys || echo '#{ssh_pub_key}' >> .ssh/authorized_keys
        grep -q '#{ssh_pub_key}' /root/.ssh/authorized_keys || echo '#{ssh_pub_key}' >> /root/.ssh/authorized_keys
        grep -q '#{ssh_pub_key}' /home/ubuntu/.ssh/authorized_keys || echo '#{ssh_pub_key}' >> /home/ubuntu/.ssh/authorized_keys
      SHELL
    end
    s.inline = <<-SHELL
      #{shell_script}
      # hadoop
      grep -q '#{hadoop_ssh_pub_key}' .ssh/authorized_keys || echo '#{hadoop_ssh_pub_key}' >> .ssh/authorized_keys
      grep -q '#{hadoop_ssh_pub_key}' /home/ubuntu/.ssh/authorized_keys || echo '#{hadoop_ssh_pub_key}' >> /home/ubuntu/.ssh/authorized_keys
      grep -q '#{hadoop_ssh_pub_key}' /root/.ssh/authorized_keys || echo '#{hadoop_ssh_pub_key}' >> /root/.ssh/authorized_keys

      [ ! -e .ssh/id_rsa ] && echo '#{hadoop_ssh_pem_key}' >> .ssh/id_rsa
      [ ! -e /home/ubuntu/.ssh/id_rsa ] && echo '#{hadoop_ssh_pem_key}' >> /home/ubuntu/.ssh/id_rsa
      [ ! -e /root/.ssh/id_rsa ] && echo '#{hadoop_ssh_pem_key}' >> /root/.ssh/id_rsa

      [ ! -e .ssh/config ] && echo "Host *\n  StrictHostKeyChecking no\n  UserKnownHostsFile /dev/null\n" >> .ssh/config
      [ ! -e /home/ubuntu/.ssh/config ] && echo "Host *\n  StrictHostKeyChecking no\n  UserKnownHostsFile /dev/null\n" >> /home/ubuntu/.ssh/config
      [ ! -e /root/.ssh/config ] && echo "Host *\n  StrictHostKeyChecking no\n  UserKnownHostsFile /dev/null\n" >> /root/.ssh/config

      grep -q 'ubuntu ALL=(ALL) NOPASSWD:ALL' /etc/sudoers.d/90-cloud-init-users || echo 'ubuntu ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers.d/90-cloud-init-users
      grep -q 'vagrant ALL=(ALL) NOPASSWD:ALL' /etc/sudoers.d/90-cloud-init-users || echo 'vagrant ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers.d/90-cloud-init-users
      sudo sed -i -e "\\#PasswordAuthentication yes# s#PasswordAuthentication yes#PasswordAuthentication no#g" /etc/ssh/sshd_config
      sudo service ssh restart
    SHELL
  end

  (101..103).each do |i|
    config.vm.define "data#{(i % 100)}" do |node|
      node.vm.hostname = "data#{(i % 100)}"
      node.vm.network :private_network, ip: "192.168.56.#{i}"
      node.vm.provider 'virtualbox' do |v|
        v.memory = 1024
      end
    end
  end

  config.vm.define 'master' do |node|
    node.vm.hostname = 'master'
    node.vm.network :private_network, ip: '192.168.56.100'
    node.vm.provider 'virtualbox' do |v|
      v.memory = 1024
    end

    node.vm.provision 'ansible' do |ansible|
      # ansible.verbose = 'v'
      ansible.playbook = 'main.yml'
      ansible.limit = 'all'
      ansible.inventory_path = 'hosts'

      ansible.raw_arguments = [
        '--connection=paramiko'
      ]
    end
  end
end
