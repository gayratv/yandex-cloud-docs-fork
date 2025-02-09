# Creating a VM from a set of disks

You can create a VM from existing disks. The disks must reside in one of the availability zones and not be added to other VMs.

{% include [disk-auto-delete](../../_includes_service/disk-auto-delete.md) %}

To create a VM from a set of disks:

{% list tabs %}

- Management console

  To create a VM:

  1. In the [management console]({{ link-console-main }}), select the folder to create your VM in.
  1. In the list of services, select **{{ compute-name }}**.
  1. Click **Create VM**.
  1. Under **Basic parameters**:

      * Enter a name and description for the VM. Naming requirements:

          {% include [name-format](../../../_includes/name-format.md) %}

          {% include [name-fqdn](../../../_includes/compute/name-fqdn.md) %}

      * Select the [availability zone](../../../overview/concepts/geo-scope.md) to host the VM in.

  1. Under **Image/boot disk selection**, select one of the [images](../../operations/images-with-pre-installed-software/get-list.md).

  1. (optional) Under **Disks**, click **Add disk**:
      * Enter the disk name.
      * Select the [disk type](../../concepts/disk.md#disks_types).
      * Specify the desired block size.
      * Specify the necessary disk size.
      * (optional) Enable the **Delete with the VM** option if you need to automatically delete the disk when deleting the VM it will be attached to.
      * Under **Content**, choose **Disk** and select the disk to attach. If you don't have a disk, [create](../disk-create/empty.md) one.
      * Click **Add**.

  1. (optional) Under **File storages**, attach the [file storage](../../concepts/filesystem.md) and enter the device name.

  1. Under **Computing resources**:
      * Choose a [platform](../../concepts/vm-platforms.md).
      * Specify the [guaranteed share](../../../compute/concepts/performance-levels.md) and number of vCPUs and RAM you need.
      * If necessary, make your VM [preemptible](../../concepts/preemptible-vm.md).
      * (optional) Enable a [software-accelerated network](../../concepts/software-accelerated-network.md).

  1. Under **Network settings**:
      * Specify the subnet ID or select a [cloud network](../../../vpc/concepts/network.md#network) from the list.
          If you don't have a network, click **Create network** to create one:
          * In the window that opens, enter the network name and folder to host the network.
          * (optional) To automatically create subnets, select the **Create subnets** option.
          * Click **Create**.
          Each network must have at least one [subnet](../../../vpc/concepts/network.md#subnet). If there is no subnet, create one by selecting **Add subnet**.
      * In the **Public IP** field, choose a method for assigning an IP address:
          * **Auto**: Assign a random IP address from the {{ yandex-cloud }} IP pool. In this case, you can enable [protection from DDoS attacks](../../../vpc/ddos-protection/index.md) using the option below.
          * **List**: Select a public IP address from the list of previously reserved static addresses. For more information, see [{#T}](../../../vpc/operations/set-static-ip.md).
          * **No address**: Don't assign a public IP address.
      * In the **Internal address** field, select the method for assigning internal addresses: **Auto** or **Manual**.
      * (optional) Create a record for the VM in the [DNS zone](../../../dns/concepts/dns-zone.md). Expand the **DNS settings for internal addresses** section, click **Add record** and specify the zone, FQDN and TTL for the record. For more information, see [Yandex Cloud DNS integration with Yandex Compute Cloud](../../../dns/concepts/compute-integration.md).
      * Select the [appropriate security groups](../../../vpc/concepts/security-groups.md) (if there is no corresponding field, all incoming and outgoing traffic will be allowed for the VM).

  1. Under **Access**, specify the data required to access the VM:

      * (optional) Select or create a [service account](../../../iam/concepts/users/service-accounts.md). By using a service account, you can flexibly configure access rights for your resources.
      * Enter the username in the **Login** field.

          {% note alert %}

          Don't use the username `root` or other names reserved by the operating system. To perform operations that require superuser permissions, use the command `sudo`.

          {% endnote %}

      * In the **SSH key** field, paste the contents of the [public key file](../../operations/vm-connect/ssh.md#creating-ssh-keys).
      * If required, grant access to the [serial console](../../operations/serial-console/index.md).

  1. Click **Create VM**.

  The virtual machine appears in the list. When a VM is being created, it is assigned an [IP address](../../../vpc/concepts/address.md) and [hostname](../../../vpc/concepts/address.md#fqdn) (FQDN).

- CLI

  {% include [cli-install](../../../_includes/cli-install.md) %}

  {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

  1. View the description of the CLI command for creating a VM:

      ```
      $ yc compute instance create --help
      ```

  1. Get a list of disks in the default folder:

      {% include [compute-disk-list](../../../_includes/compute/disk-list.md) %}

  1. Select the identifier (`ID`) or name (`NAME`) of the necessary disks.

  1. Create a VM in the default folder:

      ```
      $ yc compute instance create \
          --name first-instance \
          --zone ru-central1-a \
          --network-interface subnet-name=default-a,nat-ip-version=ipv4 \
          --use-boot-disk disk-name=first-disk \
          --attach-disk disk-name=second-disk \
          --ssh-key ~/.ssh/id_rsa.pub
      ```

      This command creates the VM:

      - Named `first-instance`.
      - In the `ru-central1-a` availability zone.
      - In the `default-a` subnet.
      - With a public IP address and two disks.

      To create a VM without a public IP, remove the `--public-ip` flag.

      {% include [name-format](../../../_includes/name-format.md) %}

      {% include [name-fqdn](../../../_includes/compute/name-fqdn.md) %}

      To specify whether to delete the disk when deleting the VM, set the `--auto-delete` flag:

      ```
      yc compute instance create \
      --name first-instance \
      --zone ru-central1-a \
      --network-interface subnet-name=default-a,nat-ip-version=ipv4 \
      --use-boot-disk disk-name=first-disk,auto-delete=yes \
      --attach-disk disk-name=second-disk,auto-delete=yes \
      --ssh-key ~/.ssh/id_rsa.pub
      ```

- API

  Use the [Create](../../api-ref/Instance/create.md) method for the `Instance` resource.

- Terraform

  If you don't have Terraform yet, [install it and configure the {{ yandex-cloud }} provider](../../../tutorials/infrastructure-management/terraform-quickstart.md#install-terraform).

  To create a VM from a set of disks:

  1. In the configuration file, describe the parameters of resources that you want to create:

       {% note info %}

       If you already have suitable resources, such as a cloud network and subnet, you don't need to describe them again. Use their names and IDs in the appropriate parameters.

       {% endnote %}

     * `yandex_compute_instance`: Description of the [VM](../../concepts/vm.md):
       * `name`: VM name.
       * `platform_id`: The [platform](../../concepts/vm-platforms.md).
       * `resources`: The number of vCPU cores and the amount of RAM available to the VM. The values must match the selected [platform](../../concepts/vm-platforms.md).
       * `boot_disk`: Boot disk settings. Specify the disk ID. If you have no boot disk available, specify the [ID of a public image](../images-with-pre-installed-software/get-list.md) using the `image_id` parameter.
       * `secondary_disk`: Secondary disk to attach to the VM. Specify the ID of the secondary disk. If you don't have a disk, [create](../disk-create/empty.md) one.
       * `network_interface`: Network settings. Specify the ID of the selected subnet. To automatically assign a public IP address to the VM, set `nat = true`.
       * `metadata`: In the metadata, pass the public key for accessing the VM via SSH. For more information, see [{#T}](../../concepts/vm-metadata.md).
     * `yandex_vpc_network`: Description of the [cloud network](../../../vpc/concepts/network.md#network).
     * `yandex_vpc_subnet`: Description of the [subnet](../../../vpc/concepts/network.md#network) that the VM will be connected to.

     Example configuration file structure:

     ```
     resource "yandex_compute_instance" "vm-1" {
     
       name        = "vm-from-disks"
       platform_id = "standard-v3"
     
       resources {
         cores  = <number of vCPU cores>
         memory = <RAM in GB>
       }
     
       boot_disk {
         initialize_params {
           disk_id = "<boot disk ID>"
         }
       }
     
       secondary_disk {
         disk_id = "<secondary disk ID>"
       }
     
       network_interface {
         subnet_id = "${yandex_vpc_subnet.subnet-1.id}"
         nat       = true
       }
     
       metadata = {
         ssh-keys = "<username>:<SSH key contents>"
       }
     }
     
     resource "yandex_vpc_network" "network-1" {
       name = "network1"
     }
     
     resource "yandex_vpc_subnet" "subnet-1" {
       name       = "subnet1"
       zone       = "<availability zone>"
       network_id = "${yandex_vpc_network.network-1.id}"
     }
     ```

     For more information about the resources you can create using Terraform, see the [provider documentation](https://www.terraform.io/docs/providers/yandex/index.html).

  2. Make sure that the configuration files are correct.

     1. In the command line, go to the directory where you created the configuration file.

     2. Run the check using the command:

        ```
        $ terraform plan
        ```

     If the configuration is described correctly, the terminal displays a list of created resources and their parameters. If there are errors in the configuration, Terraform points them out.

  3. Deploy the cloud resources.

     1. If the configuration doesn't contain any errors, run the command:

        ```
        $ terraform apply
        ```

     2. Confirm that you want to create the resources.

     Afterwards, all the necessary resources are created in the specified folder. You can check resource availability and their settings in the [management console]({{ link-console-main }}).

{% endlist %}

