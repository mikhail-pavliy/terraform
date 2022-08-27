# Terraform как инструмент для декларативного описания инфраструктуры // ДЗ 
Для выполнения ДЗ нам понадобиться доступ к ЯО

1. Создадим отдельный каталог для размещения ресурсов, созданных при выполнении ДЗ. 

2. Настройка рабочей станции

2.1. Установим git 
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ sudo apt install git
```
2.2. Установим terraform согласно инсьрукции с оф.сайта
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ wget -O- https://apt.releases.hashicorp.com/gpg | \
    gpg --dearmor | \
    sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ gpg --no-default-keyring \
    --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    --fingerprint
gpg: /home/mikhail/.gnupg/trustdb.gpg: создана таблица доверия
/usr/share/keyrings/hashicorp-archive-keyring.gpg
-------------------------------------------------
pub   rsa4096 2020-05-07 [SC]
      E8A0 32E0 94D8 EB4E A189  D270 DA41 8C88 A321 9F7B
uid         [ неизвестно ] HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
sub   rsa4096 2020-05-07 [E]
```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ sudo apt-get install terraform
```

3. Создадим папку ```terraform``` и файл ```.gitignore```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ mkdir terraform
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform$ nano .gitignore
```
и скопируем следующее содержимое в файл ```.gitignore```
```ruby
# Created by https://www.toptal.com/developers/gitignore/api/terraform
# Edit at https://www.toptal.com/developers/gitignore?templates=terraform

### Terraform ###
# Local .terraform directories
**/.terraform/*

# .tfstate files
*.tfstate
*.tfstate.*

# Crash log files
crash.log

# Exclude all .tfvars files, which are likely to contain sentitive data, such as
# password, private keys, and other secrets. These should not be part of version
# control as they are data points which are potentially sensitive and subject
# to change depending on the environment.
#
*.tfvars
*.auto.tfvars

# Ignore override files as they are usually used to override resources locally and so
# are not checked in
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Include override files you do wish to add to version control using negated pattern
# !example_override.tf

# Include tfplan files to ignore the plan output of command: terraform plan -out=tfplan
# example: *tfplan*

# Ignore CLI configuration files
.terraformrc
terraform.rc

# End of https://www.toptal.com/developers/gitignore/api/terraform
```

4. Провайдер. Создадим файл ```provider.tf``` со следующим содержимым:

```ruby
provider "yandex" {
  token     = var.yc_token
  cloud_id  = var.yc_cloud
  folder_id = var.yc_folder
}

terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}
```
5. Создадим файл ```variables.tf``` со следующим содержимым:
```ruby
variable "yc_cloud" {
  type = string
  description = "Yandex Cloud ID"
}

variable "yc_folder" {
  type = string
  description = "Yandex Cloud folder"
}

variable "yc_token" {
  type = string
  description = "Yandex Cloud OAuth token"
}

variable "db_password" {
  description = "MySQL user pasword"
}
```
6. Создадим файл ```wp.auto.tfvars```

```ruby
yc_cloud  = "b1gf5768rgabjbptan7a"
yc_folder = "b1gb8haadbndninaj928"
yc_token = "jlkdsflgjoisgoskljgs"
db_password = "password"
```
7. Произведем инициализацию командой ```terraform init```

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of yandex-cloud/yandex...
- Installing yandex-cloud/yandex v0.77.0...
- Installed yandex-cloud/yandex v0.77.0 (self-signed, key ID E40F590B50BB8E40)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
8. Создадим виртуальную сеть. для этого будем использовать файл ```network.tf``` со следующим содержимым:

```ruby
resource "yandex_vpc_network" "wp-network" {
  name = "wp-network"
}

resource "yandex_vpc_subnet" "wp-subnet-a" {
  name = "wp-subnet-a"
  v4_cidr_blocks = ["10.2.0.0/16"]
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.wp-network.id
}

resource "yandex_vpc_subnet" "wp-subnet-b" {
  name = "wp-subnet-b"
  v4_cidr_blocks = ["10.3.0.0/16"]
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.wp-network.id
}

resource "yandex_vpc_subnet" "wp-subnet-c" {
  name = "wp-subnet-c"
  v4_cidr_blocks = ["10.4.0.0/16"]
  zone           = "ru-central1-c"
  network_id     = yandex_vpc_network.wp-network.id
}
```
9. Проверим как работает данный манифест, выполним команду:

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform apply --auto-approve

Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

```
10. Виртуальные машины. Создадим манифесты для хостов, где будет разворачиваться WordPress. 
Создадим файл ```wp-app.tf``` со следующим содержимым:
```ruby
resource "yandex_compute_instance" "wp-app-1" {
  name = "wp-app-1"
  zone = "ru-central1-a"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd80viupr3qjr5g6g9du"
    }
  }

  network_interface {
    # Указан id подсети default-ru-central1-a
    subnet_id = yandex_vpc_subnet.wp-subnet-a.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/yc.pub")}"
  }
}

resource "yandex_compute_instance" "wp-app-2" {
  name = "wp-app-2"
  zone = "ru-central1-b"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd80viupr3qjr5g6g9du"
    }
  }

  network_interface {
    # Указан id подсети default-ru-central1-b
    subnet_id = yandex_vpc_subnet.wp-subnet-b.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/yc.pub")}"
  }
}
```
11. Выполним команду и убедимся, что виртуальные машины созданы успешно: ```terraform apply --auto-approve```

```mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform apply --auto-approve

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

12. Балансировщик трафика. Итак, виртуальные машины у нас есть, теперь мы можем создать балансировщик, который будет перенаправлять на них пользовательский трафик.
Создадим манифест ```lb.tf``` со следующим содержимым:

```ruby
resource "yandex_lb_target_group" "wp_tg" {
  name      = "wp-target-group"

  target {
    subnet_id = yandex_vpc_subnet.wp-subnet-a.id
    address   = yandex_compute_instance.wp-app-1.network_interface.0.ip_address
  }

  target {
    subnet_id = yandex_vpc_subnet.wp-subnet-b.id
    address   = yandex_compute_instance.wp-app-2.network_interface.0.ip_address
  }
}

resource "yandex_lb_network_load_balancer" "wp_lb" {
  name = "wp-network-load-balancer"

  listener {
    name = "wp-listener"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }

  attached_target_group {
    target_group_id = yandex_lb_target_group.wp_tg.id

    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/health"
      }
    }
  }
}
````
13. Запустим команду ```terraform apply --auto-approve``` применения манифестов и убедимся, что виртуальные машины созданы успешно:

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform apply --auto-approve

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

```
14. База данных. Необходимо создать в YC облачный кластер MySQL и опишем для этого соответствующий манифест.
Назовем его ```db.tf``` Содержимое манифеста будет:
```ruby
locals {
  dbuser = tolist(yandex_mdb_mysql_cluster.wp_mysql.user.*.name)[0]
  dbpassword = tolist(yandex_mdb_mysql_cluster.wp_mysql.user.*.password)[0]
  dbhosts = yandex_mdb_mysql_cluster.wp_mysql.host.*.fqdn
  dbname = tolist(yandex_mdb_mysql_cluster.wp_mysql.database.*.name)[0]
}

resource "yandex_mdb_mysql_cluster" "wp_mysql" {
  name        = "wp-mysql"
  folder_id   = var.yc_folder
  environment = "PRODUCTION"
  network_id  = yandex_vpc_network.wp-network.id
  version     = "8.0"

  resources {
    resource_preset_id = "s2.micro"
    disk_type_id       = "network-ssd"
    disk_size          = 16
  }

  database {
    name  = "db"
  }

  user {
    name     = "user"
    password = var.db_password
    authentication_plugin = "MYSQL_NATIVE_PASSWORD"
    permission {
      database_name = "db"
      roles         = ["ALL"]
    }
  }

  host {
    zone      = "ru-central1-b"
    subnet_id = yandex_vpc_subnet.wp-subnet-b.id
    assign_public_ip = true
  }
  host {
    zone      = "ru-central1-c"
    subnet_id = yandex_vpc_subnet.wp-subnet-c.id
    assign_public_ip = true
  }
}
```
15. Запустим команду ```terraform apply --auto-approve``` 

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform apply --auto-approve

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```
16. Output-переменные. создадим файл output.tf, где укажем вывод некоторой информации, которая может нам пригодится.
Содержимое данного манифеста:
```ruby
output "load_balancer_public_ip" {
  description = "Public IP address of load balancer"
  value = yandex_lb_network_load_balancer.wp_lb.listener.*.external_address_spec[0].*.address
}

output "database_host_fqdn" {
  description = "DB hostname"
  value = local.dbhosts
}
```
17. Зпустим команду ```terraform apply --auto-approve```

```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform apply --auto-approve

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

database_host_fqdn = tolist([
  "rc1b-gof2xqthrdo9323c.mdb.yandexcloud.net",
  "rc1c-ihmu610pmgd1n9a9.mdb.yandexcloud.net",
])
load_balancer_public_ip = tolist([
  "62.84.122.27",
])

```
18. Удаление ресурсов ```terraform destroy --auto-approve```
```ruby
mikhail@mikhail-GL703VD:~/Desktop/otus/03-terraform/terraform$ terraform destroy --auto-approve

Destroy complete! Resources: 9 destroyed.

```
