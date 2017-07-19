# -*- mode: ruby -*-
# vi: set ft=ruby :

script = <<SCRIPT
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y unzip curl vim \
  apt-transport-https \
  ca-certificates \
  software-properties-common
# Download Nomad
for NOMAD_VERSION in 0.5.6 0.5.4; do
  echo "Fetching Nomad ${NOMAD_VERSION}..."
  mkdir -p /tmp/nomad-${NOMAD_VERSION}
  cd /tmp/nomad-${NOMAD_VERSION}
  curl -sSL https://releases.hashicorp.com/nomad/${NOMAD_VERSION}/nomad_${NOMAD_VERSION}_linux_amd64.zip -o nomad.zip
  echo "Installing Nomad..."
  unzip nomad.zip
  mkdir -p /usr/bin/nomad-${NOMAD_VERSION}
  install nomad /usr/bin/nomad-${NOMAD_VERSION}
done
echo "Fetching Consul..."
curl -sSL https://releases.hashicorp.com/consul/0.8.5/consul_0.8.5_linux_amd64.zip > consul.zip
mkdir -p /etc/nomad.d
chmod a+w /etc/nomad.d
# Set hostname's IP to made advertisement Just Work
#sed -i -e "s/.*nomad.*/$(ip route get 1 | awk '{print $NF;exit}') nomad/" /etc/hosts
echo "Installing Docker..."
if [[ -f /etc/apt/sources.list.d/docker.list ]]; then
    echo "Docker repository already installed; Skipping"
else
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    apt-get update
fi
DEBIAN_FRONTEND=noninteractive apt-get install -y docker-ce
# Restart docker to make sure we get the latest version of the daemon if there is an upgrade
service docker restart
# Make sure we can actually use docker as the vagrant user
usermod -aG docker vagrant
echo "Installing Consul..."
unzip /tmp/consul.zip
install consul /usr/bin/consul
(
cat <<-EOF
	[Unit]
	Description=consul agent
	Requires=network-online.target
	After=network-online.target

	[Service]
	Restart=on-failure
	ExecStart=/usr/bin/consul agent -dev
	ExecReload=/bin/kill -HUP $MAINPID

	[Install]
	WantedBy=multi-user.target
EOF
) | tee /etc/systemd/system/consul.service
systemctl enable consul.service
systemctl start consul
SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = 'bento/ubuntu-16.04' # 16.04 LTS
  config.vm.hostname = 'nomad'
  config.vm.provision 'shell', inline: script
  config.vm.provision 'docker' # Just install it

  # Increase memory for Parallels Desktop
  config.vm.provider 'parallels' do |p, _|
    p.memory = '1024'
  end

  # Increase memory for Virtualbox
  config.vm.provider 'virtualbox' do |vb|
    vb.memory = '1024'
  end

  # Increase memory for VMware
  %w[vmware_fusion vmware_workstation].each do |p|
    config.vm.provider p do |v|
      v.vmx['memsize'] = '1024'
    end
  end
end
