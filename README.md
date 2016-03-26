Ansible Role: libvirt_vm
=========

Manages deployment of libvirt/KVM virtual machines.

Requirements
------------

This role is very opinionated and expects:

 * libvirt/KVM hypervisor server
 * local virt-builder template repository
 * virt-builder template image (correctly processed by virt-sysprep before usage)
 * libvirt network in bridge mode (VMs get static IP addresses)

Role Variables
--------------

The role requires you to provide the following variables:

    libvirt_vm_template    # template ID from a local virt-builder repository
#    libvirt_vm_hostname (DEPRECATED)
#    libvirt_vm_domain (DEPRECATED)
    libvirt_vm_ip_addr
    libvirt_vm_netmask
    libvirt_vm_gateway
    libvirt_vm_mac_addr
    libvirt_vm_cpu         # number of virtual CPUs
    libvirt_vm_mem         # specify size in megabytes
    libvirt_vm_disk        # specify size and unit (eg. '20G', '1T', etc)
    libvirt_vm_osvariant   # `virt-install --os-variant list`
    libvirt_vm_hypervisor  # hostname of the libvirt/KVM hypervisor host (as specified in the Ansible inventory)

Additionally, the following default settings are defined (and can be overridden):

    # wait_for machine to be acessible on port 22, from the hypervisor's perspective
    libvirt_vm_wait_online: yes

    # network bridge to use
    libvirt_vm_bridge: "virbr0"

    # where the generated virtual disks will reside
    libvirt_vm_images_path: "/var/lib/libvirt/images"

    # root password (default: disabled)
    libvirt_vm_root_password: "disabled"

    # virt-builder repository location (can also be remote, see the virt-builder documentation)
    libvirt_vm_virtbuilder_source: "/var/lib/libvirt/templates/index.asc"

    # virt-builder repository fingerprint
    libvirt_vm_virtbuilder_fingerprint: "unused"

    # additional options to be passed to virt-builder
    libvirt_vm_virtbuilder_additional_opts: "--no-check-signature"

If you sign your virt-builder repository with GPG, be sure to include the fingerprint and remove the --no-check-signature from the additional options.

Dependencies
------------

None.

Example Playbook
----------------

    ---
    - name: Deploy a libvirt/KVM virtual machine
      hosts: localhost
      gather_facts: no
      become: no
    
      roles:
    
        - role: libvirt_vm
          libvirt_vm_template:   "centos-7.1"
          libvirt_vm_hostname:   "webserver"
          libvirt_vm_domain:     "example.com"
          libvirt_vm_ip_addr:    "192.168.1.10"
          libvirt_vm_mac_addr:   "02:00:00:00:00:01"
          libvirt_vm_cpu:        "2"
          libvirt_vm_mem:        "2048"
          libvirt_vm_disk:       "20G"
          libvirt_vm_osvariant:  "rhel7.0"
          libvirt_vm_hypervisor: "kvmhost"

You need to pass a tag to `ansible-playbook` for the operation to be executed (launch, terminate, start, stop):

    $ ansible-playbook -i inventory -t launch site.yml
