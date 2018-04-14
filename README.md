Deploy ManageIQ in oVirt
==================================================

The `oVirt.manageiq` role downloads a ManageIQ/CloudForms QCOW image and deploys it into oVirt/Red Hat Virtualization (RHV).

The role also enables you to create a virtual machine and attach the ManageIQ disk, then wait for the ManageIQ system to initialize, and register oVirt as an infrastructure provider.

Requirements
------------

* oVirt has to be 4.0.4 or higher.
* Ansible has to be 2.5 or higher.
* [ovirt-imageio](http://www.ovirt.org/develop/release-management/features/storage/image-upload/) must be installed and running.
* [oVirt Python SDK version 4](https://pypi.python.org/pypi/ovirt-engine-sdk-python/4.2.4).

Additionally, perform the following checks to ensure the required processes are running.
* Check whether `ovirt-imageio-proxy` is running on the engine:

 ```
systemct status ovirt-imageio-proxy
```

* Check whether `ovirt-imageio-daemon` is running on the hosts:

 ```
systemct status ovirt-imageio-daemon
```

You will also require the CA certificate of the engine. To do this, configure the `ovirt_ca` variable with the path to the CA certificate.

Role Variables
--------------

QCOW variables:

| Name          | Default value                                            |  Description                                                 |
|---------------|----------------------------------------------------------|--------------------------------------------------------------|
| miq_qcow_url  | http:////releases.manageiq.org/manageiq-ovirt-fine-1.qc2 | The URL of the ManageIQ QCOW image. |
| miq_image_path | /tmp/ovirt_image_data | The path where the qcow2 image will be downloaded. |
| miq_qcow_checksum | sha256:b3644e8ac75af9663d19372e21b8a0273d68e54bfd515<br/>518321d3102d08daebd | Checksum of the qcow2 image file. It's used to validate the downloaded file.  |

Engine login variables:

| Name                | Default value     |  Description                            |
|---------------------|-------------------|-----------------------------------------|
| engine_user         | UNDEF             | The user to access the engine.          |
| engine_password     | UNDEF             | The password of the 'engine_user'.      |
| engine_fqdn         | UNDEF             | The FQDN of the engine.                 |
| engine_ca           | UNDEF             | The path to the engine's CA certificate.|

Virtual machine variables:

| Name                  | Default value       |  Description                                                   |
|-----------------------|---------------------|----------------------------------------------------------------|
| miq_vm_name           | manageiq_fine       | The name of the ManageIQ virtual machine.                      |
| miq_vm_cluster        | Default             | The cluster of the virtual machine.                            |
| miq_vm_memory         | 6GiB                | The virtual machine's system memory.                           |
| miq_vm_cpu            | 2                   | The number of virtual machine CPU cores.                       |
| miq_vm_os             | rhel_7x64           | The virtual machine operating system.                          |
| miq_vm_root_password  | `miq_app_password`  | The root password for the virtual machine.                     |
| miq_vm_cloud_init     | UNDEF               | The cloud init dictionary to be passed to the virtual machine. |

Virtual machine main disks variables (e.g. operating system):

| Name                | Default value     |  Description                            |
|---------------------|-------------------|-----------------------------------------|
| miq_vm_disk_name    | `miq_vm_name`     | The name of the virtual machine disk.   |
| miq_vm_disk_storage | UNDEF             | The target storage domain of the disk.  |
| miq_vm_disk_size    | 50GiB             | The virtual machine disk size.          |
| miq_vm_disk_interface | virtio          | The virtual machine disk interface type.|
| miq_vm_disk_format  | cow               | The format of the virtual machine disk. |

Virtual machine extra disks (e.g. database, log, tmp): a dict named
`miq_vm_disks` allows to describe each of the extra disks (see example
playbook). For each disk, the following attributes can be set:

| Name      | Default value |  Description                                                         |
|-----------|---------------|----------------------------------------------------------------------|
| name      | UNDEF         | The name of the virtual machine disk. i                              |
| size      | UNDEF         | The virtual machine disk size (`XXGiB`).                             |
| interface | UNDEF         | The virtual machine disk interface type (`virtio` or `virtio_scsi`). |
| format    | UNDEF         | The format of the virtual machine disk (`raw` or `cow`).             |

Virtual machine NICs variables:

| Name                | Default value     |  Description                                         |
|---------------------|-------------------|------------------------------------------------------|
| miq_vm_nics         | {'name': 'nic1', 'profile_name': 'ovirtmgmt', 'interaface': 'virtio'} | List of dictionaries that defines the virtual machine network interfaces. |

The item in `miq_vm_nics` list of can contain following attributes:

| Name               | Default value  |                                              |
|--------------------|----------------|----------------------------------------------|
| name               | UNDEF          | The name of the network interface.           |
| interface          | UNDEF          | Type of the network interface.               |
| mac_address        | UNDEF          | Custom MAC address of the network interface, by default it's obtained from MAC pool. |
| network            | UNDEF          | Logical network which the VM network interface should use. If network is not specified, then Empty network is used. |
| profile            | UNDEF          | Virtual network interface profile to be attached to VM network interface. |

ManageIQ variables:

| Name               | Default value       |  Description                                                               |
|--------------------|---------------------|----------------------------------------------------------------------------|
| miq_app_username   | admin               | The username used to login to ManageIQ.                                    |
| miq_app_password   | smartvm             | The password of user specific in username used to login to ManageIQ.       |
| miq_username       | admin               | Alias of `miq_app_username` for backward compatibility.                    |
| miq_password       | smartvm             | Alias of `miq_app_password` for backward compatibility.                    |
| miq_db_username    | root                | The username to connect to the database.                                   |
| miq_db_password    | `miq_app_password`  | The password of user specific in username used to connect to the database. |
| miq_region         | 0                   | The ManageIQ region created in the database.                               |
| miq_company        | My Company          | The company name of the appliance.                                         |
| miq_disabled_roles | []                  | List of ManageIQ roles to disable on the appliance.                        |
| miq_enabled_roles  | []                  | List of ManageIQ roles to enable on the appliance.                         |


RHV provider and RHV metrics variables:

| Name                  | Default value     |  Description                                           |
|-----------------------|-------------------|--------------------------------------------------------|
| miq_rhv_provider_name | RHV provider      | Name of the RHV provider to be displayed in ManageIQ.  |
| metrics_fqdn          | UNDEF             | FQDN of the oVirt/RHV metrics.                         |
| metrics_user          | UNDEF             | The user to connection to metrics server.              |
| metrics_password      | UNDEF             | The password of the `metrics_user` .                   |
| metrics_port          | UNDEF             | Port to connect to oVirt/RHV metrics.                  |
| metrics_db_name       | ovirt_engine_history | Database name of the oVirt engine metrics database. |

Dependencies
------------

No.

Example Playbook
----------------

Note that for passwords you should use Ansible vault.

```yaml
    - name: Deploy ManageIQ to oVirt engine
      hosts: localhost
      gather_facts: no

      vars:
        engine_fqdn: ovirt-engine.example.com
        engine_user: admin@internal
        engine_password: 123456

        miq_vm_name: manageiq_fine
        miq_qcow_url: http://releases.manageiq.org/manageiq-ovirt-fine-1.qc2
        miq_vm_cluster: mycluster
        miq_vm_root_password: securepassword
        miq_vm_cloud_init:
          host_name: "{{ miq_vm_name }}"
        miq_vm_disks:
          database:
            name: "{{ miq_vm_name }}_Disk2"
            size: 10GiB
            interface: virtio
            format: raw
          log:
            name: "{{ miq_vm_name }}_Disk3"
            size: 10GiB
            interface: virtio
            format: cow
          tmp:
            name: "{{ miq_vm_name }}_Disk4"
            size: 10GiB
            interface: virtio
            format: cow
        miq_disabled_roles:
          - smartstate
        miq_enabled_roles:
          - notifier
          - ems_metrics_coordinator
          - ems_metrics_collector
          - ems_metrics_processor

      roles:
        - oVirt.manageiq
```

License
-------

Apache License 2.0
