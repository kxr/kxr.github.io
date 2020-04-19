---
layout: post
title: OpenShift 4 UPI Installation on Libvirt/KVM
---	

I am writing this blog post to document the procedure of installing OpenShift 4 UPI (User Provisioned Infrastructure) on KVM/Libvirt using the Baremetal procedure as reference. This can be a useful reference if you are looking to:

* Setup OpenShift 4 on your laptop, workstation or a server.
* Learn/Teach/Demo the OpenShift 4 UPI Installation procedure.

All you need is an internet connected host that has enough CPU/Memory and virtualization support for KVM.

If you follow along, by the end of this post, you should have a working OpenShift 4 cluster with 3 master nodes, 2 worker nodes and a VM acting as the external load balancer.

### Update 19Apr2020:

* Created a script to automate this procedure. See: https://github.com/kxr/ocp4_setup_upi_kvm
* Demo:
[![asciicast](https://asciinema.org/a/bw6Wja2vBLrAkpKHTV0yGeuzo.svg)](https://asciinema.org/a/bw6Wja2vBLrAkpKHTV0yGeuzo)



## Prerequisites and Assumptions

* Make sure that the host has a stable and fast connection to the internet. This is important as during the boostrap process, all the control plane nodes download roughly 4-5 GB of images from the container registry (quay.io).

* It is assumed that Virtualization support enabled in BIOS of the Host/Hypervisor and libvirt/KVM is already setup with the standard configuration. You are free however, to pick any one of libvirt's network for the VMs of this OpenShift cluster. Make sure that `libvirt-daemon-driver-network` is installed)

* It is also assumed that which ever libvirt's network you pick, has dhcp enabled on it. By default the "default" network has dhcp enabled. If for some reason, you want to create a segregated network for this OpenShift install, you can easily do it by:

  ~~~ bash
  # Pick a random subnet octet (192.168.XXX.0) for this network
  NET_OCT="133"
  /usr/bin/cp /usr/share/libvirt/networks/default.xml /tmp/new-net.xml
  sed -i "s/default/ocp-${NET_OCT}/" /tmp/new-net.xml
  sed -i "s/virbr0/ocp-$NET_OCT/" /tmp/new-net.xml
  sed -i "s/122/${NET_OCT}/g" /tmp/new-net.xml
  virsh net-define /tmp/new-net.xml
  virsh net-autostart ocp-${NET_OCT}
  virsh net-start ocp-${NET_OCT}
  systemctl restart libvirtd
  ~~~

* We need a DNS server and we will be using dnsmasq. You have two options <font color='red'>(pick either one)</font>:

  1. **Use NetworkManager's embedded dnsmasq**

      This is preferable if you are running this on your laptop with dynamic interfaces getting IP/DNS from DHCP. dnsmasq in NetworkManager is not enabled by defualt. If you don't have this enabled and want to use this mode, you can simply enable it by:

     ~~~ bash
     echo -e "[main]\ndns=dnsmasq" > /etc/NetworkManager/conf.d/nm-dns.conf
     systemctl restart NetworkManager
     ~~~
      To read more about this feature I recommend reading this [Fedora Magazine post by Clark](https://fedoramagazine.org/using-the-networkmanagers-dnsmasq-plugin/).

  2. **Setup a separate dnsmasq server on the host**

      This is preferable if the network on the host is not being managed by NetworkManager. To setup dnsmasq run the following commands (adjust according to your environment):

     ~~~ bash
     yum -y install dnsmasq
     for x in $(virsh net-list --name); do virsh net-info $x | awk '/Bridge:/{print "except-interface="$2}'; done > /etc/dnsmasq.d/except-interfaces.conf
     sed -i '/^nameserver/i nameserver 127.0.0.1' /etc/resolv.conf
     systemctl restart dnsmasq
     systemctl enable dnsmasq
     ~~~

## Installation Procedure

I am following the [OpenShift 4 UPI Installation documentation for bare-metal](https://docs.openshift.com/container-platform/4.2/installing/installing_bare_metal/installing-bare-metal.html) with some minor tweaks to get it working on KVM, so I recommend going through it once, if you haven't already.

**Note:** *I have intentionally tried to make this copy-paste friendly, to make it easily reproducible. This is my usual style of documenting setup procedures. It is expected that you start in single bash session and run these stream of commands one by one. Each block section is copy-paste-able at a time. If you find any breaking points or have any suggestions, please drop me a line.*

### 1- Download Files and Setup Environment

  * Start by switching to the root user if you are not already root:

    ~~~ bash
    sudo -i
    ~~~

  * Make sure you have screen installed.

    ~~~ bash
    yum -y install screen
    ~~~

  * Create and switch to a new directory to keep things clean:

    ~~~ bash
    mkdir ocp4 && cd ocp4
    ~~~

  * All the VMs will be created on the libvirt's network that you pick. By default libvirt has only one "default" network. You can find out libvirt's networks by running `virsh net-list`

    ~~~ bash
    VIR_NET="default"
    ~~~

  * Based on the libvirt's network selected, we need to find out the Network and IP address of libvirt's bridge interface.

    ~~~ bash
    HOST_NET=$(ip -4 a s $(virsh net-info $VIR_NET | awk '/Bridge:/{print $2}') | awk '/inet /{print $2}')
    HOST_IP=$(echo $HOST_NET | cut -d '/' -f1)
    ~~~

  * Set the dnsmasq configuration directory.

    If you are using NetworkManager's embedded dnsmasq, set it to "/etc/NetworkManager/dnsmasq.d".
    If you are using a separate dnsmasq installed on the host set it to "/etc/dnsmasq.d".

    ~~~ bash
    DNS_DIR="/etc/NetworkManager/dnsmasq.d"
    ~~~

  * **<font color='orange'>[DNS CHECK POINT]</font>**

    At this point its worth double checking that your DNS is working as expected or the installation will fail later. Most of the installation I have seen failing is because of the DNS.

      1. Make sure that the hosts/hypervisor's dns is pointing to the local dnsmasq. The first entry in /etc/resolv.conf should be "nameserver 127.0.0.1".

      2. Make sure that any entry in /etc/hosts is forward and reverse resolvable by libvirt/kvm. You can test this by adding a test record in /etc/hosts:

         `echo "1.2.3.4 test.local" >> /etc/hosts`

         and then restart libvirtd so it picks up the hosts file:

         `systemctl restart libvirtd`

         and finally check if the forward and reverse lookup works:

         `dig test.local @${HOST_IP}`

         `dig -x 1.2.3.4 @${HOST_IP}`

         verify that you get answers in both the above dig queries.

      3. Make sure that any entry in the dnsmasq.d is also picked up by libvirt/kvm. You can test this by adding a test srv record:

         `echo "srv-host=test.local,yayyy.local,2380,0,10" > ${DNS_DIR}/temp-test.conf`

         and then reload dnsmasq (if you are using separate dnsmasq service then restart dnsmasq, if its NetworkManager's dnsmasq then reload NetworkManager):

         `# If using separate dnsmasq service

         `systemctl restart dnsmasq

         `# If using NetworkManager's dnsmasq

         `systemctl reload NetworkManager`

         and finally test that both libvirt and your host can resolve the srv record:

         `dig srv test.local`

         `dig srv test.local @${HOST_IP}`

         verify that you get the srv answer (yayy.local) in both the above dig queries.

      4. Clean up: remove the test entry in /etc/hosts and delete the ${DNS_DIR}/temp-test.conf file.

  * Pick a base domain for your cluster:

    ~~~ bash
    BASE_DOM="local"
    ~~~

  * Pick a cluster name:

    ~~~ bash
    CLUSTER_NAME="ocp41"
    ~~~

  * Pick your SSH public key (file):

    ~~~ bash
    SSH_KEY="/root/.ssh/id_rsa.pub"
    ~~~

  * Download your pull secret from [Red Hat OpenShift Cluster Manager](https://cloud.redhat.com/openshift/install/metal/user-provisioned) and load into a variable. You can also copy paste the pull secret. The variable PULL_SEC should have your pull secret without any newlines.

    **NOTE:** If you are copy-pasting the value of pull secret, its crucial to use single quotes (not double quotes).

    ~~~ bash
    # If loading from downloaded file
    PULL_SEC=$(cat </download/path>/pull-secret)
    # If copy-pasting, use single quotes
    PULL_SEC='<paste-pull-secret>'
    ~~~

  * Download the RHCOS Install kernel and initramfs and generate the treeinfo. *(I am doing this instead of using the RHCOS install ISO because it allows us to pass kernel arguments easily via `virt-install --extra-args` instead of typing it manually in the VM's console)*

    ~~~ bash
    mkdir rhcos-install
    wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.2/4.2.0/rhcos-4.2.0-x86_64-installer-kernel -O rhcos-install/vmlinuz
    wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.2/4.2.0/rhcos-4.2.0-x86_64-installer-initramfs.img -O rhcos-install/initramfs.img
    
    cat <<EOF > rhcos-install/.treeinfo
    [general]
    arch = x86_64
    family = Red Hat CoreOS
    platforms = x86_64
    version = 4.2.0
    [images-x86_64]
    initrd = initramfs.img
    kernel = vmlinuz
    EOF
    ~~~

  * Download the RHCOS bios image:

    ~~~ bash
    wget https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.2/4.2.0/rhcos-4.2.0-x86_64-metal-bios.raw.gz
    ~~~

  * Download the RHEL guest image for KVM. We will use this to setup an external load balancer using haproxy. Visit [RHEL download page](https://access.redhat.com/downloads/content/69/ver=/rhel---7/7.7/x86_64/product-software) (login required) and copy the download link of Red Hat Enterprise Linux KVM Guest Image (right-click on "Download Now" and copy link location). Then use wget to download the qcow2 image (its important to put the link in quotes):

    ~~~ bash
    wget "<rhel_kvm_image_url>" -O /var/lib/libvirt/images/${CLUSTER_NAME}-lb.qcow2
    ~~~

  * Set the Red Hat Customer Portal credentials. We will need this when we register the RHEL guest to install haproxy.

    ~~~ bash
    RHNUSER='<your-rhn-user-name>'
    RHNPASS='<your-rhn-password>'
    ~~~

  * Download and extract the OpenShift client and install binaries:

    ~~~ bash
    wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.2.0/openshift-install-linux-4.2.0.tar.gz
    wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.2.0/openshift-client-linux-4.2.0.tar.gz
    
    tar xf openshift-client-linux-4.2.0.tar.gz
    tar xf openshift-install-linux-4.2.0.tar.gz
    rm -f README.md
    ~~~

  * Create the installation directory for the OpenShift installer:

    ~~~ bash
    mkdir install_dir
    ~~~

  * Generate the install-config.yaml:

    ~~~ bash
	cat <<EOF > install_dir/install-config.yaml
    apiVersion: v1
    baseDomain: ${BASE_DOM}
    compute:
    - hyperthreading: Disabled
      name: worker
      replicas: 0
    controlPlane:
      hyperthreading: Disabled
      name: master
      replicas: 3
    metadata:
      name: ${CLUSTER_NAME}
    networking:
      clusterNetworks:
      - cidr: 10.128.0.0/14
        hostPrefix: 23
      networkType: OpenShiftSDN
      serviceNetwork:
      - 172.30.0.0/16
    platform:
      none: {}
    pullSecret: '${PULL_SEC}'
    sshKey: '$(cat $SSH_KEY)'
    EOF
	~~~

  * Generate the ignition files:

    ~~~ bash
	./openshift-install create ignition-configs --dir=./install_dir
    ~~~

  * Start python's webserver, serving the current directory in screen:

    ~~~ bash
    # Pick a port that you want to listen on
    WEB_PORT="1234"
    screen -S ${CLUSTER_NAME} -dm bash -c "python3 -m http.server ${WEB_PORT}"
    ~~~

  * Make sure that the VMs can access the host on the web port. Ignore this if you don't have iptables/firewalld turned on.

    ~~~ bash
    # If using firewalld
    firewall-cmd --add-source=${HOST_NET}
    firewall-cmd --add-port=${WEB_PORT}/tcp
    
    # If using iptables
    iptables -I INPUT -p tcp -m tcp --dport ${WEB_PORT} -s ${HOST_NET} -j ACCEPT
    ~~~

    Note: We are intentionally not opening the port permanently. We only need it in the next section when we spawn up coreos vms.

  * **<font color='orange'>[CHECK POINT]</font>**

    At this point, make sure that our current directory (ocp4) is being served by python. Also make sure that you can access the ignition (ing) and image (img) files. Simply visit http://localhost:1234 on the host using a browser or curl. Also double check the URLs we are going to pass to the RHCOS installer kernel in the next three commands. Make sure that those URLs are reachable from inside the VMs.

### 2- Create the Red Hat CoreOS and Load Balancer VMs

  * Spawn the bootstrap node:

    ~~~ bash
    virt-install --name ${CLUSTER_NAME}-bootstrap \
      --disk size=50 --ram 16000 --cpu host --vcpus 4 \
      --os-type linux --os-variant rhel7 \
      --network network=${VIR_NET} --noreboot --noautoconsole \
      --location rhcos-install/ \
      --extra-args "nomodeset rd.neednet=1 coreos.inst=yes coreos.inst.install_dev=vda coreos.inst.image_url=http://${HOST_IP}:${WEB_PORT}/rhcos-4.2.0-x86_64-metal-bios.raw.gz coreos.inst.ignition_url=http://${HOST_IP}:${WEB_PORT}/install_dir/bootstrap.ign"
    ~~~

  * Spawn three master nodes:

    ~~~ bash
    for i in {1..3}
    do
    virt-install --name ${CLUSTER_NAME}-master-${i} \
    --disk size=50 --ram 16000 --cpu host --vcpus 4 \
    --os-type linux --os-variant rhel7 \
    --network network=${VIR_NET} --noreboot --noautoconsole \
    --location rhcos-install/ \
    --extra-args "nomodeset rd.neednet=1 coreos.inst=yes coreos.inst.install_dev=vda coreos.inst.image_url=http://${HOST_IP}:${WEB_PORT}/rhcos-4.2.0-x86_64-metal-bios.raw.gz coreos.inst.ignition_url=http://${HOST_IP}:${WEB_PORT}/install_dir/master.ign"
    done
    ~~~

  * Spawn two worker nodes:

    ~~~ bash
    for i in {1..2}
    do
      virt-install --name ${CLUSTER_NAME}-worker-${i} \
      --disk size=50 --ram 8192 --cpu host --vcpus 4 \
      --os-type linux --os-variant rhel7 \
      --network network=${VIR_NET} --noreboot --noautoconsole \
      --location rhcos-install/ \
      --extra-args "nomodeset rd.neednet=1 coreos.inst=yes coreos.inst.install_dev=vda coreos.inst.image_url=http://${HOST_IP}:${WEB_PORT}/rhcos-4.2.0-x86_64-metal-bios.raw.gz coreos.inst.ignition_url=http://${HOST_IP}:${WEB_PORT}/install_dir/worker.ign"
    done
    ~~~

  * Setup the RHEL guest image that we downloaded for the load balancer.

    ~~~ bash
    virt-customize -a /var/lib/libvirt/images/${CLUSTER_NAME}-lb.qcow2 \
      --uninstall cloud-init \
      --ssh-inject root:file:$SSH_KEY --selinux-relabel \
      --sm-credentials "${RHNUSER}:password:${RHNPASS}" \
      --sm-register --sm-attach auto --install haproxy
    ~~~

  * Create the load balancer VM.

    ~~~ bash
    virt-install --import --name ${CLUSTER_NAME}-lb \
      --disk /var/lib/libvirt/images/${CLUSTER_NAME}-lb.qcow2 --memory 1024 --cpu host --vcpus 1 \
      --network network=${VIR_NET} --noreboot --noautoconsole
    ~~~

  * **<font color='orange'>[CHECK POINT]</font>**

    We have just created 7 virtual machines in KVM using virt-install. virt-install should power-off these VMs once it successfully finishes. Wait for VMs to be properly installed and powered off. You can use the following command to watch the status of the VMs:

    `watch "virsh list --all | grep '${CLUSTER_NAME}-'"`

    If the coreos VMs are note getting turned off, it most likely means that coreos was unable to fetch the image and ignition files. Open the VMs conosle to see what is going on.

### 3- Setup DNS and Load balancing

  * We will tell dnsmasq to treat our cluster domain <cluster-name>.<base-domain> as local.

    ~~~ bash
    echo "local=/${CLUSTER_NAME}.${BASE_DOM}/" > ${DNS_DIR}/${CLUSTER_NAME}.conf
    ~~~

  * Start up all the VMs.

    ~~~ bash
    for x in lb bootstrap master-1 master-2 master-3 worker-1 worker-2
    do
      virsh start ${CLUSTER_NAME}-$x
    done
    ~~~

  * Find the IP and MAC address of the bootstrap VM. Add DHCP reservation (so the VM always get the same IP) and an entry in /etc/hosts:

    ~~~ bash
    IP=$(virsh domifaddr "${CLUSTER_NAME}-bootstrap" | grep ipv4 | head -n1 | awk '{print $4}' | cut -d'/' -f1)
    MAC=$(virsh domifaddr "${CLUSTER_NAME}-bootstrap" | grep ipv4 | head -n1 | awk '{print $2}')
    virsh net-update ${VIR_NET} add-last ip-dhcp-host --xml "<host mac='$MAC' ip='$IP'/>" --live --config
    echo "$IP bootstrap.${CLUSTER_NAME}.${BASE_DOM}" >> /etc/hosts
    ~~~

  * Find the IP and MAC address of the master VMs. Add DHCP reservation (so the VMs always gets the same IP), and an entry in /etc/hosts. We will also add the corresponding etcd host and SRV record:

    ~~~ bash
    for i in {1..3}
    do
      IP=$(virsh domifaddr "${CLUSTER_NAME}-master-${i}" | grep ipv4 | head -n1 | awk '{print $4}' | cut -d'/' -f1)
      MAC=$(virsh domifaddr "${CLUSTER_NAME}-master-${i}" | grep ipv4 | head -n1 | awk '{print $2}')
      virsh net-update ${VIR_NET} add-last ip-dhcp-host --xml "<host mac='$MAC' ip='$IP'/>" --live --config
      echo "$IP master-${i}.${CLUSTER_NAME}.${BASE_DOM}" \
      "etcd-$((i-1)).${CLUSTER_NAME}.${BASE_DOM}" >> /etc/hosts
      echo "srv-host=_etcd-server-ssl._tcp.${CLUSTER_NAME}.${BASE_DOM},etcd-$((i-1)).${CLUSTER_NAME}.${BASE_DOM},2380,0,10" >> ${DNS_DIR}/${CLUSTER_NAME}.conf
    done
    ~~~

  * Find the IP and MAC address of the worker VMs. Add DHCP reservation (so the VM always get the same IP) and an entry in /etc/hosts.

    ~~~ bash
    for i in {1..2}
    do
       IP=$(virsh domifaddr "${CLUSTER_NAME}-worker-${i}" | grep ipv4 | head -n1 | awk '{print $4}' | cut -d'/' -f1)
       MAC=$(virsh domifaddr "${CLUSTER_NAME}-worker-${i}" | grep ipv4 | head -n1 | awk '{print $2}')
       virsh net-update ${VIR_NET} add-last ip-dhcp-host --xml "<host mac='$MAC' ip='$IP'/>" --live --config
       echo "$IP worker-${i}.${CLUSTER_NAME}.${BASE_DOM}" >> /etc/hosts
    done
    ~~~

  * Find the IP and MAC address of the load balancer VM. Add DHCP reservation (so the VM always get the same IP) and an entry in /etc/hosts. We will also add the api and api-int host records as they should point to this load balancer.

    ~~~ bash
    LBIP=$(virsh domifaddr "${CLUSTER_NAME}-lb" | grep ipv4 | head -n1 | awk '{print $4}' | cut -d'/' -f1)
    MAC=$(virsh domifaddr "${CLUSTER_NAME}-lb" | grep ipv4 | head -n1 | awk '{print $2}')
    virsh net-update ${VIR_NET} add-last ip-dhcp-host --xml "<host mac='$MAC' ip='$LBIP'/>" --live --config
    echo "$LBIP lb.${CLUSTER_NAME}.${BASE_DOM}" \
    "api.${CLUSTER_NAME}.${BASE_DOM}" \
    "api-int.${CLUSTER_NAME}.${BASE_DOM}" >> /etc/hosts
    ~~~

  * Create the wild-card DNS record and point it to the load balancer:

    ~~~ bash
    echo "address=/apps.${CLUSTER_NAME}.${BASE_DOM}/${LBIP}" >> ${DNS_DIR}/${CLUSTER_NAME}.conf
    ~~~

  * Configure load balancing (haproxy). We will add frontend/backend configuration in haproxy to point the required ports (6443, 22623, 80, 443) to their corresponding endpoints:

    ~~~ bash
    # Optional,
    # Just to make sure SSH access is clear
    
    ssh-keygen -R lb.${CLUSTER_NAME}.${BASE_DOM}
    ssh-keygen -R $LBIP
    ssh -o StrictHostKeyChecking=no lb.${CLUSTER_NAME}.${BASE_DOM} true
    ~~~

    ~~~ bash
    ssh lb.${CLUSTER_NAME}.${BASE_DOM} <<EOF
    
    # Allow haproxy to listen on custom ports
    semanage port -a -t http_port_t -p tcp 6443
    semanage port -a -t http_port_t -p tcp 22623
    
    echo '
    global
      log 127.0.0.1 local2
      chroot /var/lib/haproxy
      pidfile /var/run/haproxy.pid
      maxconn 4000
      user haproxy
      group haproxy
      daemon
      stats socket /var/lib/haproxy/stats
    
    defaults
      mode tcp
      log global
      option tcplog
      option dontlognull
      option redispatch
      retries 3
      timeout queue 1m
      timeout connect 10s
      timeout client 1m
      timeout server 1m
      timeout check 10s
      maxconn 3000
    # 6443 points to control plan
    frontend ${CLUSTER_NAME}-api *:6443
      default_backend master-api
    backend master-api
      balance source
      server bootstrap bootstrap.${CLUSTER_NAME}.${BASE_DOM}:6443 check
      server master-1 master-1.${CLUSTER_NAME}.${BASE_DOM}:6443 check
      server master-2 master-2.${CLUSTER_NAME}.${BASE_DOM}:6443 check
      server master-3 master-3.${CLUSTER_NAME}.${BASE_DOM}:6443 check
    
    # 22623 points to control plane
    frontend ${CLUSTER_NAME}-mapi *:22623
      default_backend master-mapi
    backend master-mapi
      balance source
      server bootstrap bootstrap.${CLUSTER_NAME}.${BASE_DOM}:22623 check
      server master-1 master-1.${CLUSTER_NAME}.${BASE_DOM}:22623 check
      server master-2 master-2.${CLUSTER_NAME}.${BASE_DOM}:22623 check
      server master-3 master-3.${CLUSTER_NAME}.${BASE_DOM}:22623 check
    
    # 80 points to worker nodes
    frontend ${CLUSTER_NAME}-http *:80
      default_backend ingress-http
    backend ingress-http
      balance source
      server worker-1 worker-1.${CLUSTER_NAME}.${BASE_DOM}:80 check
      server worker-2 worker-2.${CLUSTER_NAME}.${BASE_DOM}:80 check
    
    # 443 points to worker nodes
    frontend ${CLUSTER_NAME}-https *:443
      default_backend infra-https
    backend infra-https
      balance source
      server worker-1 worker-1.${CLUSTER_NAME}.${BASE_DOM}:443 check
      server worker-2 worker-2.${CLUSTER_NAME}.${BASE_DOM}:443 check
    ' > /etc/haproxy/haproxy.cfg
    
    systemctl start haproxy
    systemctl enable haproxy
    EOF
    ~~~

  * **<font color='orange'>[CHECK POINT]</font>**

    Make sure haproxy in the load balancer VM is up and running and listening on the desired ports:

    `ssh lb.${CLUSTER_NAME}.${BASE_DOM} systemctl status haproxy`

    `ssh lb.${CLUSTER_NAME}.${BASE_DOM} netstat -nltupe | grep ':6443\|:22623\|:80\|:443'`

  * Reload NetworkManager and Libvirt for DNS entries to be loaded properly:

    ~~~ bash
    systemctl reload NetworkManager
    systemctl restart libvirtd
    ~~~

    If you are using separate dnsmasq, restart it:

    ~~~ bash
    systemctl restart dnsmasq
    ~~~

### 4- BootStrap OpenShift 4 Cluster

  * Start the OpenShift installation, stopping at bootrap completion:

    ~~~ bash
    ./openshift-install --dir=install_dir wait-for bootstrap-complete
    ~~~

  * **<font color='orange'>[CHECK POINT]</font>**

    The OpenShift bootstrap process has started and the control plane will now download a bunch of container images from the registry (quay.io). If internet connection is slow, this will take a long time and might timeout.

    From a new terminal on the host, ssh into the bootstrap node and watch the bootkube.service service:

    `ssh core@<bootstrap-node> journalctl -b -f -u bootkube.service`

    Wait until the installer gives you the following message:

    ~~~ bash
    INFO API v1.13.4+8560dd6 up
    INFO Waiting up to 30m0s for bootstrapping to complete...
    DEBUG Bootstrap status: complete
    INFO It is now safe to remove the bootstrap resources
    ~~~

  * At this point the bootstrap should have successfully finished and the openshift-install should have told you to that its now safe to remove the bootstrap.

    You can login to your openshift cluster and make sure that all the nodes are in a ready state:

    ~~~ bash
    export KUBECONFIG=install_dir/auth/kubeconfig
    ./oc get nodes
    ~~~

  * Once you have verified that all cluster nodes (3 master and 2 worker) are in a ready state, lets remove the boostrap entries from our load balancer (haproxy).

    ~~~ bash
    ssh lb.${CLUSTER_NAME}.${BASE_DOM} <<EOF
    sed -i '/bootstrap\.${CLUSTER_NAME}\.${BASE_DOM}/d' /etc/haproxy/haproxy.cfg
    systemctl restart haproxy
    EOF
    ~~~

  * Delete the bootstrap VM

    ~~~ bash
    virsh destroy ${CLUSTER_NAME}-bootstrap
    virsh undefine ${CLUSTER_NAME}-bootstrap --remove-all-storage
    ~~~

  * We can close the screen session running the python web server.

    ~~~ bash
    screen -S ${CLUSTER_NAME} -X quit
    ~~~

  * Since we do not have a RWX storage type at the moment, we will patch image registry configuration to use "EmptyDir" for storage.

    **NOTE:** This command will fail early on, as the cluster operators are being rolled out. If it fails try after a minute or so.

    ~~~ bash
    ./oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'`
    ~~~

  * At this point OpenShift's cluster operators are being rolled out. Wait for the cluster operators to be fully available.

    ~~~ bash
    watch "./oc get clusterversion; echo; ./oc get clusteroperators"
    ~~~

  * **<font color='orange'>[CHECK POINT]</font>**

    When all the cluster operators are fully available, it should look some thing like this:

    ~~~ bash
    # ./oc get clusteroperators
    NAME                               VERSION AVAILABLE PROGRESSING DEGRADED SINCE
    authentication                     4.1.9 True False False 11m
    cloud-credential                   4.1.9 True False False 24m
    cluster-autoscaler                 4.1.9 True False False 24m
    console                            4.1.9 True False False 13m
    dns                                4.1.9 True False False 22m
    image-registry                     4.1.9 True False False 2m58s
    ingress                            4.1.9 True False False 16m
    kube-apiserver                     4.1.9 True False False 20m
    kube-controller-manager            4.1.9 True False False 20m
    kube-scheduler                     4.1.9 True False False 20m
    machine-api                        4.1.9 True False False 23m
    machine-config                     4.1.9 True False False 22m
    marketplace                        4.1.9 True False False 16m
    monitoring                         4.1.9 True False False 13m
    network                            4.1.9 True False False 22m
    node-tuning                        4.1.9 True False False 16m
    openshift-apiserver                4.1.9 True False False 18m
    openshift-controller-manager       4.1.9 True False False 20m
    openshift-samples                  4.1.9 True False False 6m13s
    operator-lifecycle-manager         4.1.9 True False False 20m
    operator-lifecycle-manager-catalog 4.1.9 True False False 20m
    service-ca                         4.1.9 True False False 23m
    service-catalog-apiserver          4.1.9 True False False 17m
    service-catalog-controller-manager 4.1.9 True False False 18m
    storage                            4.1.9 True False False 17m
    
    
    # ./oc get clusterversion
    NAME    VERSION AVAILABLE PROGRESSING SINCE STATUS
    version 4.1.9   True      False       3m46s Cluster version is 4.1.9
    ~~~

### 5- Finish Installation and Access the Cluster

  * At this point you are pretty much done. Finish the installation by running:

    ~~~ bash
    ./openshift-install --dir=install_dir wait-for install-complete
    ~~~

  * **<font color='orange'>[CHECK POINT]</font>**

    The opshift-install should give you an output similar to:

    ~~~ bash
    INFO Waiting up to 30m0s for the cluster at https://api.ocp41.local:6443 to initialize...
    INFO Waiting up to 10m0s for the openshift-console route to be created...
    INFO Install complete!
    INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/knaeem/Desktop/ocp4/install_dir/auth/kubeconfig'
    INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocp41.local
    INFO Login to the console with user: kubeadmin, password: XXXX-XXXX-XXXX-XXXX
    ~~~

  * You can now login to the cluster using the kubeadmin user from the command-line:

    ~~~ bash
    KUBE_PASS=$(cat install_dir/auth/kubeadmin-password)
    ./oc login -u kubeadmin -p $KUBE_PASS
    ~~~

  * Access the cluster from the browser:

    ~~~ bash
    xdg-open https://console-openshift-console.apps.${CLUSTER_NAME}.${BASE_DOM}
    ~~~

    ![OpenShift 4 Screenshot]({{ "/" | relative_url }}public/media/openshift-4-upi-install-libvirt-kvm/Screenshot_2019-08-17_Login_Red_Hat_OpenShift_Container_Platform.png)

    ![OpenShift 4 Screenshot]({{ "/" | relative_url }}public/media/openshift-4-upi-install-libvirt-kvm/Screenshot_2019-08-17_Cluster_Status_Red_Hat_OpenShift_Container_Platform.png)

    ![OpenShift 4 Screenshot]({{ "/" | relative_url }}public/media/openshift-4-upi-install-libvirt-kvm/Screenshot_2019-08-17_Nodes_Red_Hat_OpenShift_Container_Platform.png)

    ![OpenShift 4 Screenshot]({{ "/" | relative_url }}public/media/openshift-4-upi-install-libvirt-kvm/Screenshot_2019-08-17_Projects_Red_Hat_OpenShift_Container_Platform.png)

## Known Issues

  * Make sure that virtualization support is enabled in BIOS. If its not enabled, libvirt will probably not complain but the CPU emulation will make the VMs super slow and the CoreOS VMs won't boot properly.

  * If you are getting timed out while waiting for Kubernetes API and the installer (`./openshift-install --dir=install_dir wait-for bootstrap-complete`) throws the following error:

    `FATAL waiting for Kubernetes API: context deadline exceeded`

    Check the following:

    * Make sure that you have good internet speed. If the internet is slow it might cause the installer to timeout before the images get downloaded.

    * Make sure the first nameserver on the host in /etc/resolv.conf is 127.0.0.1, and that NetworkManager restart is not resetting it to something else.

    * Login into the one of the master node `ssh core@master-1.<cluster-name>.<base-domain>` using the ssh key you selected during the setup. Then become root (sudo -i) and run (critctl ps). You should see a container named "discover". See the logs of this container (crictl logs <container-id>) to get a clue on why things are not starting up.

    * If have figured out and fixed the issue, its safe re run this command (./openshift-install --dir=install_dir wait-for bootstrap-complete). You don't have to scratch every thing and re-run.

