a2init = <<SCRIPT
apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y unzip
sysctl -w vm.max_map_count=262144
sysctl -w vm.dirty_expire_centisecs=20000
curl -fsSL https://packages.chef.io/files/current/automate/latest/chef-automate_linux_amd64.zip -o /tmp/chef-automate_linux_amd64.zip
unzip -qod /tmp /tmp/chef-automate_linux_amd64.zip
chmod +x /tmp/chef-automate
/tmp/chef-automate deploy --accept-terms-and-mlsa --product automate
cat > /vagrant/automate-eas.toml << EOF
[event_gateway]
  [event_gateway.v1]
    [event_gateway.v1.sys]
      [event_gateway.v1.sys.service]
        disable_frontend_tls = true
EOF
chef-automate config patch /vagrant/automate-eas.toml
echo "Server is up and running. Please log in at https://chef-automate-test.net/"
echo 'You may log in using credentials provided below:'
cat /home/vagrant/automate-credentials.toml
cat > data-collector-token.toml <<EOF
[auth_n.v1.sys.service]
a1_data_collector_token = "pjTlwgQuAeg8Kl9uTsucBsJArf8="
EOF
chef-automate config patch data-collector-token.toml
cp -fn /home/vagrant/automate-credentials.toml /vagrant/
sudo chef-automate external-cert show > /vagrant/automate.cert
SCRIPT

clients = <<SCRIPT
sudo yum install nano -y
useradd hab
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
echo '192.168.33.199  chef-automate-test.net' >> /etc/hosts
echo CHEF_LICENSE='accept' >> /etc/environment
echo HAB_LICENSE='accept' >> /etc/environment
source /etc/environment
curl https://raw.githubusercontent.com/habitat-sh/habitat/master/components/hab/install.sh | sudo bash
hab license accept
mkdir -p /hab/cache/ssl
cp -f /vagrant/automate.cert /hab/cache/ssl/
sudo hab pkg install core/hab-sup
mkdir -p /hab/user/config-baseline/config /hab/user/chef-cis-sample-linux-inspec/config
cat > /hab/user/config-baseline/config/user.toml <<EOF
interval = 300
splay = 30
splay_first_run = 0
run_lock_timeout = 1800
log_level = "warn"

[data_collector]
enable = "true"
server_url = "https://chef-automate-test.net/data-collector/v0/"
token = "pjTlwgQuAeg8Kl9uTsucBsJArf8="
EOF
cat > /hab/user/chef-cis-sample-linux-inspec/config/user.toml <<EOF
interval = 300
splay = 30
splay_first_run = 0
log_level = 'warn'

[chef_license]
acceptance = "accept-no-persist"

[automate]
enable = true
server_url = "https://chef-automate-test.net"
token = "pjTlwgQuAeg8Kl9uTsucBsJArf8="
user = "admin"
EOF
nohup hab run --auto-update --listen-gossip 0.0.0.0:9638 --listen-http 0.0.0.0:9631 --event-stream-application="effortless" --event-stream-environment="test" --event-stream-site="DC1" --event-stream-url="chef-automate-test.net:4222" --event-stream-token="pjTlwgQuAeg8Kl9uTsucBsJArf8=" > /var/log/hab-sup.log 2>&1&
sleep 30
hab svc load chef/chef-cis-sample-linux-inspec
SCRIPT

Vagrant.configure('2') do |config|
  # Setup Automate server
  config.vm.define 'a2' do |a2|
    a2.vm.provider 'virtualbox' do |v|
      v.memory = 4096
      v.cpus = 2
      v.name = 'a2'
    end
    a2.vm.box = 'bento/ubuntu-16.04'
    a2.vm.provision 'shell', inline: a2init
    a2.vm.hostname = 'chef-automate-test.net'
    a2.vm.network 'private_network', ip: '192.168.33.199'
  end
  # Setup client systems
  %w(system1 system2).each do |client|
    config.vm.define client do |initclient|
      initclient.vm.box = 'bento/centos-8'
      initclient.vm.provision 'shell', inline: clients
      initclient.vm.hostname = "#{client}"
      initclient.vm.network 'private_network', ip: "192.168.33.2#{rand(50)}"
    end
  end
end
