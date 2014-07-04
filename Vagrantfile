# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'securerandom'

# Some things we'll need
rootpassword = SecureRandom.urlsafe_base64(20)

# These only need to be set if using the Rackspace Provider
cloudUsername       = "username"
api_key             = "apikey"
flavor              = /4GB/
image               = /Ubuntu 12.04/i
public_key_path     = "/path/to/key.pub"
rackspace_region    = :dfw


# In here we specify the nodes as well as how many, and what starting IP to use in the last octet
# If provisioning multiples, ensure you have enough IPs between nodes
nodes = {
    'rpcs-controller' => [2, 101],
    'rpcs-compute' => [1, 120],
    'rpcs-chef'  => [1, 100],
}

###########
# You shouldn't need to edit below here
# Thar be dragons of terrible bash/ruby mashup... you've been warned.
###########

#####
# Here's a bit of script that everything gets.
####
$commonscript = <<COMMONSCRIPT
# Set verbose
set -v

# Set exit on error
set -e

# Silly Ubuntu 12.04 doesn't have the
# --stdin option in the passwd utility
echo root:secrete | chpasswd
COMMONSCRIPT

####
# This script builds Chef
# It also uses knife-bootstrap to provision the other nodes
####

$chefscript = <<CHEFSCRIPT
#!/bin/bash
# chef.sh

# Set verbose
set -v

# Set exit on error
set -e

# Take care of some details
export DEBIAN_FRONTEND=noninteractive
MY_IP=$(ifconfig eth1 | awk '/inet addr/ {split ($2,A,":"); print A[2]}')
MY_IP2=$(ifconfig eth0 | awk '/inet addr/ {split ($2,A,":"); print A[2]}')

echo "${MY_IP2}     rpcs-chef-01.cook.book" >> /etc/hosts

wget https://raw.github.com/rcbops/support-tools/master/chef-install/install-chef-server.sh -O /tmp/install-chef-server.sh

chmod +x /tmp/install-chef-server.sh

export CHEF_URL="https://rpcs-chef-01.cook.book:443"

/tmp/install-chef-server.sh

cd /root

git clone https://github.com/rcbops/chef-cookbooks.git

cd chef-cookbooks

git checkout v4.2.2
git submodule init
git submodule sync
git submodule update

knife cookbook upload -a -o cookbooks

knife role from file roles/*rb

cat << EOF >> /tmp/rpcv422.json
{
    "name": "rpcv422",
    "description": "Rackspace Private Cloud v4.2.2",
    "cookbook_versions": {},
    "json_class": "Chef::Environment",
    "chef_type": "environment",
    "default_attributes": {},
    "override_attributes": {
        "nova": {
            "libvirt": {
                "virt_type": "qemu",
                "vncserver_listen": "0.0.0.0"
            },
            "network": {
                "provider": "neutron"
            }
        },
        "neutron": {
            "ovs": {
                "provider_networks": [
                    {
                        "label": "ph-eth3",
                        "bridge": "br-eth3"
                    }
                ],
                "network_type": "gre",
                "network": "neutron",
                "external_bridge": ""
            }
        },
        "cinder": {
            "services": {
                "volume": {
                    "network": "cinder"
                }
            }
        },        
        "mysql": {
            "allow_remote_root": true,
            "root_network_acl": "%"
        },
        "osops_networks": {
            "nova": "192.168.236.0/24",
            "public": "192.168.236.0/24",
            "management": "192.168.236.0/24",
            "neutron": "192.168.240.0/24",
            "cinder": "192.168.248.0/24"
        },
        "vips": {
            "rabbitmq-queue": "192.168.236.50",
            "ceilometer-api": "192.168.236.51",
            "ceilometer-central-agent": "192.168.236.51",
            "cinder-api": "192.168.236.51",
            "glance-api": "192.168.236.51",
            "glance-registry": "192.168.236.51",
            "heat-api": "192.168.236.51",
            "heat-api-cfn": "192.168.236.51",
            "heat-api-cloudwatch": "192.168.236.51",
            "horizon-dash": "192.168.236.51",
            "horizon-dash_ssl": "192.168.236.51",
            "keystone-admin-api": "192.168.236.51",
            "keystone-internal-api": "192.168.236.51",
            "keystone-service-api": "192.168.236.51",
            "nova-api": "192.168.236.51",
            "nova-ec2-public": "192.168.236.51",
            "nova-novnc-proxy": "192.168.236.51",
            "nova-xvpvnc-proxy": "192.168.236.51",
            "swift-proxy": "192.168.236.51",
            "neutron-api": "192.168.236.51",
            "mysql-db": "192.168.236.52",
            "config": {
                "192.168.236.50": {
                    "vrid": 1,
                    "network": "public"
                },
                "192.168.236.51": {
                    "vrid": 2,
                    "network": "public"
                },
                "192.168.236.52": {
                    "vrid": 3,
                    "network": "public"
                }
            }
        }
    }
}
EOF

knife environment from file /tmp/rpcv422.json

ssh-keygen -t rsa -N "" -f /root/.ssh/id_rsa

sudo apt-get install -y expect

CHEFSCRIPT

####
# This script gets run from the chef server for each node that needs to be provisioned
####

$buildscript = <<BUILDSCRIPT

knife bootstrap rpcs-controller-01.cook.book --environment rpcv422 --run-list 'role[ha-controller1],role[single-network-node]' || while ! knife ssh "name:rpcs-controller-01.cook.book" "chef-client"; do echo "chef-client failed, retrying"; sleep 5; done

knife bootstrap rpcs-controller-02.cook.book --environment rpcv422 --run-list 'role[ha-controller2],role[single-network-node]' || while ! knife ssh "name:rpcs-controller-02.cook.book" "chef-client"; do echo "chef-client failed, retrying"; sleep 5; done

knife ssh "name:rpcs-controller-01.cook.book" "chef-client"

BUILDSCRIPT

$sshscript = <<sshscript

ssh-keyscan ${hostname}.cook.book >> /root/.ssh/known_hosts

expect<<EOF
spawn ssh-copy-id ${hostname}.cook.book
expect "root@${hostname}.cook.book's password:"
send "$secrete\n"
expect eof
EOF

sshscript

Vagrant.configure("2") do |config|
    # Virtual Box
    config.vm.box = "ubuntu-server-12.04.3-lts-x86_64"
    config.vm.box_url = "http://public.thornelabs.net/ubuntu-server-12.04.4-lts-x86_64.box"
    config.vm.synced_folder ".", "/vagrant", type: "nfs"

    # VMware Fusion
    config.vm.provider "vmware_fusion" do |vmware, override|
        override.vm.box = "ubuntu-server-12.04.3-lts-x86_64.vmware"
        override.vm.box_url = "http://public.thornelabs.net/ubuntu-server-12.04.4-lts-x86_64.vmware.box"
        override.vm.synced_folder ".", "/vagrant", type: "nfs"

        # Fusion Performance Hacks
        vmware.vmx["logging"] = "FALSE"
        vmware.vmx["MemTrimRate"] = "0"
        vmware.vmx["MemAllowAutoScaleDown"] = "FALSE"
        vmware.vmx["mainMem.backing"] = "swap"
        vmware.vmx["sched.mem.pshare.enable"] = "FALSE"
        vmware.vmx["snapshot.disabled"] = "TRUE"
        vmware.vmx["isolation.tools.unity.disable"] = "TRUE"
        vmware.vmx["unity.allowCompostingInGuest"] = "FALSE"
        vmware.vmx["unity.enableLaunchMenu"] = "FALSE"
        vmware.vmx["unity.showBadges"] = "FALSE"
        vmware.vmx["unity.showBorders"] = "FALSE"
        vmware.vmx["unity.wasCapable"] = "FALSE"
    end

    # Rackspace Cloud
    config.vm.provider "rackspace" do |rs, override|
        override.vm.box = "dummy"
        override.vm.box_url = "https://github.com/mitchellh/vagrant-rackspace/raw/master/dummy.box"
        rs.username        = cloudUsername
        rs.api_key         = api_key
        rs.flavor          = flavor
        rs.image           = image
        rs.rackspace_region = rackspace_region
    end

    if Vagrant.has_plugin?("vagrant-hostmanager")
        config.hostmanager.enabled = true
        config.hostmanager.manage_host = true
        config.hostmanager.ignore_private_ip = false
        config.hostmanager.include_offline = false
    else
        puts "You need to install vagrant-hostmanager. Run: vagrant plugin install vagrant-hostmanager"
        abort
    end

#    config.ssh.private_key_path = "/Users/bunchc/.ssh/id_rsa"
    hostips = Hash.new
    
    nodes.each do |prefix, (count, ip_start)|
        count.times do |i|

            hostname = "%s-%02d" % [prefix, (i+1)]
            config.vm.define "#{hostname}" do |box|
                box.vm.hostname = "#{hostname}.cook.book"

                box.vm.provision :shell, :inline => $commonscript
                # Set IPs if using a local provider (vbox / fusion)
                box.vm.network "private_network", ip: "192.168.236.#{ip_start+i}", :netmask => "255.255.255.0"
                box.vm.network "private_network", ip: "192.168.240.#{ip_start+i}", :netmask => "255.255.255.0"
                box.vm.network "private_network", ip: "192.168.244.#{ip_start+i}", :netmask => "255.255.255.0"
                box.vm.network "private_network", ip: "192.168.248.#{ip_start+i}", :netmask => "255.255.255.0"
                if prefix == "rpcs-chef"
                    
                    # Build the Chef server
                    box.vm.provision :shell, :inline => $chefscript

                    # Get all the hosts
                    box.vm.provision :hostmanager
                    
                    # Add all the SSH Keys to Chef
                    nodes.each do |prefix, (count, ip_start)|
                        count.times do |i|
                            hostname = "%s-%02d" % [prefix, (i+1)]
                            box.vm.provision :shell, :inline => "echo 'herin lies lots of little hate'
                            ssh-keyscan #{hostname}.cook.book >> /root/.ssh/known_hosts
expect<<EOF
spawn ssh-copy-id #{hostname}.cook.book
expect {
    \"Now try logging into the machine\" {
        exit
    } \"root@#{hostname}.cook.book's password:\" {
        send \"secrete\n\"    
    }
}
expect eof
EOF"
                        end
                    end
                    
                    # Use Chef to bootstrap controllers
                    box.vm.provision :hostmanager
                    box.vm.provision :shell, :inline => $buildscript

                    # Use Chef to bootstrap the compute nodes
                    nodes.each do |prefix, (count, ip_start)|
                        if prefix == "rpcs-compute"
                            count.times do |i|
                                hostname = "%s-%02d" % [prefix, (i+1)]
                                box.vm.provision :shell, :inline => "knife bootstrap #{hostname}.cook.book --environment rpcv422 --run-list 'role[single-compute]' || while ! knife ssh \"name:#{hostname}\" \"chef-client\"; do echo \"chef-client failed, retrying\"; sleep 5; done"
                            end
                        end
                    end
                end
            end
        end
    end
end
