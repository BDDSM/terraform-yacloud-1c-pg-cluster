# Terraform план кластера 1С и Postgresql в Яндекс.Облаков

План создания ресурсов для кластера 1С и кластера СУБД Postgresql в облаке провайдера Яндекс

## Возможности

* Работа в ОС семейства: Linux, Mac OS X, Windows;
* Указание различных параметров для кластеров 1С и СУБД Postgresql;
* Вывод сетевой информации по созданным кластерам, в том числе для inventory ansible;
* Использование отдельных файлов с секретами.

## Установка
* Выполнить установку утилиты terraform (не ниже версии `0.12.21`) на машину, с которой будет выполняться план:
  * Mac OS X: "`$ brew install terraform`", либо с сайта [terraform](https://www.terraform.io/downloads.html)
  * Linux: с сайта [terraform](https://www.terraform.io/downloads.html);
  * Windows: с сайта [terraform](https://www.terraform.io/downloads.html).
* Скачать проект на машину;
* При инициализации проекта (см. ниже) обязательно необходимо убедиться, что версия плагина terraform провайдера (Яндекс.Облако) не ниже `0.32.0`. Это можно сделать (после инициализации проекта, т.е. когда будет подгружен плагин провайдера) командой: "`$ terraform -v`".

## Настройка
* Заполнить параметры кластеров 1С и СУБД Postgresql в файле `settings.auto.tfvars`:
  * `v4_cidr_blocks` - тип данных `list(list)`. Список нескольких блоков адресов ipv4 для построения подсетей VPC. Минимальное количество 3, для всех зон доступности провайдера, чтобы была возможность создать отказоустойчевый кластер СУБД Postgresql. Пример в файле;
  * `fqdn_prefix` - тип данных `string`. Наименование префикса для FQDN имен машин в VPC, чтобы можно было внутри обращаться по имени;
  * `ansible_format_output` - тип даных `boolean`. True - вывод данных по имени хоста и ipv4 (внешнему) в формате подходящем для файла inventory ansible;
  * `path_to_metadatefile` - тип данных `string`. Путь к файлу с метаданными (имя пользователя, открытый ключ и т.д.) для машин кластера 1С. Пример в настройках ниже.
  * `count_app1c_servers` - тип данных `int`. Количество хостов в кластере 1С;
  * `zones` - тип данных `list`. Список зон доступности для возможности создания отказоустойчевого кластера СУБД Postgresql;
  * `instance_cores_1c` - тип данных `string`. Количество vCPU одного хоста в кластере 1С;
  * `instance_memory_1c` - тип данных `string`, Количество RAM одного хоста в кластере 1С;
  * `instance_core_fraction_1c` - тип данных `string`. Гарантированный процент доступности процессорного времени хостов в кластере 1С;
  * `os_image_id_1c` - тип данных `string`. ID образа ОС для машины. Список всех образов можно получить только с помощью CLI провайдера;
  * `instance_disk_size_1c` - тип данных `string`. Размер диска хостов в кластере 1С;
  * `instance_disk_type_1c` - тип данных `string`. Тип диска хостов в кластере 1С (актуальный список указан на сайте провайдер);
  * `instance_preemptible_1c` - тип данных `boolean`. Могут ли быть машины в кластере прерываемыми;
  * `environment` - тип данных `string`. Тип среды кластера СУБД Postgresql (актуальный список указан на сайте провайдер);
  * `pg_version` - тип данных `string`. Версия СУБД Postgresql (актуальный список указан на сайте провайдер);
  * `resource_preset_id` - тип данных `string`. Тип машины в кластере СУБД Postgresql. Тип машины указывает ее ресурсы (vCPU и RAM). Различных вариантов много (актуальный список указан на сайте провайдер);
  * `disk_type_id` - тип данных `string`. Тип диска машины в кластере СУБД Postgresql (актуальный список указан на сайте провайдер);
  * `disk_size` - тип данных `int`. Размер диска машины в кластере СУБД Postgresql;
  * `db_name` - тип данных `string`. Наименование БД в кластере. Создание хотя бы одной базы в кластере обязательно;
  * `owner` - тип данных `string`. Имя пользователя, который будет являтся владельцем создаваемой БД;
  * `lc_collate` - тип данных `string`. Локаль для порядка сортировка строк в БД;
  * `lc_type` - тип данных `string`. Локаль для классификации символов в БД;
  * `cluster_user_name` - тип данных `string`. Имя пользователя кластера СУБД.
* Создать файл с секретами `seсrets.tfvars` в корневой папке проекта:
    * `token` - тип данных `string`. Токен провайдера выданный после регистрации;
    * `cloud_id` - тип данных `string`. Идентификатор сервисного аккаунта;
    * `folder_id` - тип данных `string`. Идентификатор пространства группировки ресурсов;
    * `cluster_user_pass` - тип данных `string`. Пароль пользователя кластера СУБД.
    ```properties
    token               = "AgASg1AOnWO-AATuvlmUprlRM08e8po4"
    cloud_id            = "b1gimfp8u0"
    folder_id           = "b1et9ka3"
    cluster_user_pass   = "Ga1z"
    ```
* Создать файл с метаданными `metadata.yml` для машин кластера 1С в корнейвой папке проекта. Пример содержимого файла:
    ```properties
    #cloud-config
    users:
    - name: popka
      groups: sudo 
      sudo: ['ALL=(ALL) NOPASSWD:ALL']
      ssh-authorized-keys:
        - ssh-ed25519 AADDC3xzaC1lZDI12TE5AB9AIQO19iOku2JLApNzN1VM+2KH2UkC3lytqXKpVKWnVbRa popka@yandex.ru
    ```


## Подготовка к созданию ресурсов
### Перед выполнение проекта небходимо выполнить инициализацию и построение плана:
* Инициализация (скачивание плагина провайдера, подключение модулей и т.д.) плана создания ресурсов выполняется командой:
```sh
    $ terraform init
```
* Построение плана ресурсов, для проверки (корректности заполнения и синтаксиса) и без сохранения в отдельный файл (ключ `-out t_plan`), выполняется командой:
```sh
    $ terraform plan -var-file="secrets.tfvars"
```

## Создание ресурсов
* Запуск процесса для создания ресурсов у провайдера выполняется командой:
```sh
    $ terraform apply -var-file="secrets.tfvars"
```
  
## Удаление ресурсов
* Запуск процесса для удаления ресурсов у провайдера выполняется командой:
```sh
    $ terraform destroy -var-file="secrets.tfvars"
```

## Ресурсы
* Документация по конфигурации языка HCL - [Configuration Language](https://www.terraform.io/docs/configuration/index.html)
* Интерактивная документация для лучшего понимания terraform - [Introduction to Infrastructure as Code with Terraform](https://learn.hashicorp.com/terraform/getting-started/intro)
* Документация по ресурсам провайдера Яндекс.Облако - [Yandex.Cloud Provider](https://www.terraform.io/docs/providers/yandex/index.html)