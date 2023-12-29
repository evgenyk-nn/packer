# О Packer

Инструмент **Packer**. Позволяет удобно создавать образы виртуальных машин.

https://www.packer.io/

## Для чего нужны образы

Когда вы завершаете создание проекта, нужно перенести его из тестовой среды в рабочую. Здесь возникают две проблемы. Во-первых, чтобы решение гарантированно работало, рабочая среда должна как можно меньше отличаться от той, в которой проект создавался и тестировался. Во-вторых, проект нужно тиражировать, т.е. устанавливать и настраивать ПО на серверах или облачных платформах. Создание шаблона — образа виртуальной машины с предустановленным ПО — помогает ускорить этот процесс и уменьшить количество ошибок.

Готовые образы, доступные в [маркетплейсе](https://cloud.yandex.ru/marketplace), содержат различные версии операционных систем или наборы программ. Однако они не решают проблему быстрого масштабирования проекта. Образ с предустановленным ПО можно создать с помощью **Packer**.

## Как создать образ при помощи Packer

**Packer** работает следующим образом: вы предоставляете ему текстовый файл — **спецификацию** — с описанием сборки образа, а на выходе получаете готовый образ.

Давайте создадим образ с *Ubuntu* и веб-сервером *NGINX*. Описание образа можно составить на языке HCL ([HashiCorp Language](https://www.packer.io/docs/templates/hcl_templates)).

```hcl
source "yandex" "ubuntu-nginx" {
  token               = "<OAuth-токен>"
  folder_id           = "<идентификатор_каталога>"
  source_image_family = "ubuntu-2004-lts"
  ssh_username        = "ubuntu"
  use_ipv4_nat        = "true"
  image_description   = "my custom ubuntu with nginx"
  image_family        = "ubuntu-2004-lts"
  image_name          = "my-ubuntu-nginx"
  subnet_id           = "<идентификатор_подсети>"
  disk_type           = "network-ssd"
  zone                = "ru-central1-a"
}

build {
  sources = ["source.yandex.ubuntu-nginx"]

  provisioner "shell" {
    inline = ["sudo apt-get update -y",
              "sudo apt-get install -y nginx",
              "sudo systemctl enable nginx.service"
             ]
  }
}

```

В этом примере указано создание образа в *Yandex Cloud* с установленным *NGINX*. Параметры, такие как **`token`**, **`folder_id`**, **`image_name`**, и другие, должны быть заменены реальными значениями.

Сохраните этот файл как **`my-ubuntu-nginx.pkr.hcl`** и используйте команду:

```bash
packer build my-ubuntu-nginx.pkr.hcl

```

Остальные параметры и возможности **Packer** подробно описаны в [документации](https://www.packer.io/docs/).

## **Зачем нужно хранить шаблоны Packer**

**Packer** позволяет создавать образы для различных платформ, и хранение шаблонов в системе контроля версий (например, на GitHub) обеспечивает:

- Отслеживание изменений.
- Возможность откатываться к предыдущим версиям.
- Совместную работу нескольких разработчиков.

Подход, когда конфигурация обрабатывается как код, называется *Infrastructure as Code*.

```markdown

Этот файл `README.md` содержит краткое описание **Packer**, принципа его работы и примера использования. Пожалуйста, убедитесь, что вы заменили фиктивные значения, такие как `<OAuth-токен>`, `<идентификатор_каталога>`, `<идентификатор_подсети>`, на реальные ваши данные перед использованием шаблона `my-ubuntu-nginx.pkr.hcl`.

```


    
    вывод folder_id и subnet_id
    
    ```nix
    yc resource-manager folder list
    ➜  ~ yc resource-manager folder list
    +----------------------+---------+--------+--------+
    |          ID          |  NAME   | LABELS | STATUS |
    +----------------------+---------+--------+--------+
    | b1g6syl3ph9qh6gskmbv | default |        | ACTIVE |
    +----------------------+---------+--------+--------+
    
    There is a new yc version '0.115.0' available. Current version: '0.114.0'.
    See release notes at https://cloud.yandex.ru/docs/cli/release-notes
    You can install it by running the following command in your shell:
            $ yc components update
    
    ➜  ~ yc vpc subnet list
    +----------------------+-----------------------------------------------------------+----------------------+----------------+---------------+-----------------+
    |          ID          |                           NAME                            |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |      RANGE      |
    +----------------------+-----------------------------------------------------------+----------------------+----------------+---------------+-----------------+
    | e2lf327lgagtd94ali1c | default-ru-central1-b                                     | enp0ootf2oga7o0aad95 |                | ru-central1-b | [10.129.0.0/24] |
    | fh8fujkhjhhnfcn6vl37 | k8s-cluster-catl3273i89b9ponv5eg-service-cidr-reservation | enp0ootf2oga7o0aad95 |                | ru-central1-a | [10.96.0.0/16]  |
    | e9vfb5bgn2pmumjmsmse | default-ru-central1-a                                     | enp0ootf2oga7o0aad95 |                | ru-central1-a | [10.128.0.0/24] |
    | e9bccg6gnbt2f1gnync9 | default-ru-central1-c                                     | enp0ootf2oga7o0aad95 |                | ru-central1-a | [10.130.0.0/24] |
    | e9r9gtrfthtnb8bp35va | k8s-cluster-catl3273i89b9ponv5eg-pod-cidr-reservation     | enp0ootf2oga7o0aad95 |                | ru-central1-a | [10.112.0.0/16] |
    +----------------------+-----------------------------------------------------------+----------------------+----------------+---------------+-----------------+
    
    ➜  ~
    ```
    
    `my-ubuntu-nginx.pkr.hcl`.
    
    [`my-ubuntu-nginx.pkr.hcl`.](https://prod-files-secure.s3.us-west-2.amazonaws.com/2f230836-74eb-4ad8-b8c8-0703e897d7d5/abeeddf0-fff3-4c2f-9efe-1c704cd6c430/Untitled.txt)
    
    ```nix
    C:\Program Files\Packer>packer plugins install github.com/hashicorp/yandex
    Installed plugin github.com/hashicorp/yandex v1.1.3 in "C:/Users/ADMIN/AppData/Roaming/packer.d/plugins/github.com/hashicorp/yandex/packer-plugin-yandex_v1.1.3_x5.0_windows_amd64.exe"
    
    C:\Program Files\Packer>packer build C:\Users\ADMIN\Desktop\packer_test\my-ubuntu-nginx.pkr.hcl
    yandex.ubuntu-nginx: output will be in this color.
    
    ==> yandex.ubuntu-nginx: Creating temporary RSA SSH key for instance...
    ==> yandex.ubuntu-nginx: Using as source image: fd8k54g2t50mekbk1ie1 (name: "ubuntu-20-04-lts-v20231225", family: "ubuntu-2004-lts")
    ==> yandex.ubuntu-nginx: Use provided subnet id e9b52pmu9mhh9btlsmse
    ==> yandex.ubuntu-nginx: Creating disk...
    ==> yandex.ubuntu-nginx: Creating instance...
    ==> yandex.ubuntu-nginx: Waiting for instance with id fhm5prg9k0ngoqlek1s5 to become active...
        yandex.ubuntu-nginx: Detected instance IP: 178.154.207.62
    ==> yandex.ubuntu-nginx: Using SSH communicator to connect: 178.154.207.62
    ==> yandex.ubuntu-nginx: Waiting for SSH to become available...
    ==> yandex.ubuntu-nginx: Timeout waiting for SSH.
    ==> yandex.ubuntu-nginx: Destroying instance...
        yandex.ubuntu-nginx: Instance has been destroyed!
    ==> yandex.ubuntu-nginx: Destroying boot disk...
        yandex.ubuntu-nginx: Disk has been deleted!
    Build 'yandex.ubuntu-nginx' errored after 6 minutes 30 seconds: Timeout waiting for SSH.
    
    ==> Wait completed after 6 minutes 30 seconds
    
    ==> Some builds didn't complete successfully and had errors:
    --> yandex.ubuntu-nginx: Timeout waiting for SSH.
    
    ==> Builds finished but no artifacts were created.
    
    C:\Program Files\Packer>
    ```


