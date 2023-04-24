Vagrant.configure("2") do |config|
  config.vm.box = "rockylinux/9"
  config.vm.network "forwarded_port", guest: 8980, host: 8980, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1"

  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = 3072
    v.vmx["numvcpus"] = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    . /vagrant/versions.conf
    . /vagrant/entitlements.conf
    dnf -y install pygpgme 
    # Importing this key is necessary only until NMS-15602 is fixed
    rpm --import "https://yum.opennms.org/OPENNMS-GPG-KEY"
    rpm --import "https://packages.opennms.com/public/common/gpg.697677243260D071.key"
    curl -1sLf "https://packages.opennms.com/public/common/config.rpm.txt?distro=el&codename=9" > /tmp/opennms-common.repo
    yum-config-manager --add-repo "/tmp/opennms-common.repo"
    yum -q makecache -y --disablerepo='*' --enablerepo='opennms-common'
    rpm --import "https://packages.opennms.com/${AUTH_TOKEN}/meridian-2023/gpg.697677243260D071.key"
    curl -1sLf "https://packages.opennms.com/${AUTH_TOKEN}/meridian-2023/config.rpm.txt?distro=el&codename=9" > /tmp/opennms-meridian-2023.repo
    yum-config-manager --add-repo "/tmp/opennms-meridian-2023.repo"
    yum -q makecache -y --disablerepo='*' --enablerepo='opennms-meridian-2023'
    dnf -y install vim-enhanced nmap-ncat net-snmp net-snmp-utils rrdtool
    sed -i -e '/^view.*systemview.*included.*\.1\.3\.6\.1\.2\.1\.1/s/\.1\.3\.6\.1\.2\.1\.1$/.1/' /etc/snmp/snmpd.conf
    systemctl enable --now snmpd
    dnf -y install haveged
    dnf -y install java-11-openjdk-devel
    dnf -y install postgresql-server postgresql
    postgresql-setup initdb
    sed -i -e '/^host.*all.*all.*ident$/s/ident/trust/' /var/lib/pgsql/data/pg_hba.conf
    systemctl enable --now postgresql
    dnf -y install jrrd2 iplike-pgsql13
    dnf -y install meridian-core${MERIDIAN_VERSION:-} meridian-webapp-jetty${MERIDIAN_VERSION:-}
    hostnamectl set-hostname meridian-$(rpm -q meridian-core | awk -F- '{ print $3 }' | sed -e 's/\\./-/g')
    /opt/opennms/bin/runjava -s
    /opt/opennms/bin/install -dis
    /sbin/install_iplike-13.sh
    systemctl enable --now haveged
    systemctl enable --now opennms
    systemctl enable --now grafana-server
    sed -i -e '/^Defaults.*secure_path = /s,$,:/opt/opennms/bin,' /etc/sudoers
    /usr/bin/install -o vagrant -g vagrant -m 0600 /vagrant/dot_bash_history /home/vagrant/.bash_history
  SHELL
end
