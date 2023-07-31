# [Semaphore](https://www.ansible-semaphore.com/) - Ansible 的 UI 工具

## TLDR

1. 使用 [Vagrant](https://www.vagrantup.com/) 在 Local 建立一個 VM
2. 建立 [Ansible](https://www.ansible.com/) 的設定檔
3. 使用 [Docker Compose](https://docs.docker.com/compose/) 在 Local 運行 Ansible 的 UI 工具 [Ansible Semaphore](https://www.ansible-semaphore.com/)

## 使用 Vagrant 建立 VM

#### 安裝 VirtualBox

到 https://www.virtualbox.org/ 下載安裝 VirtualBox

#### 安裝 Vagrant

到 https://developer.hashicorp.com/vagrant/downloads 安裝作業系統對應的版本

#### 操作

- **初始化**

    *如果已經有 Vagrantfile 就不需要執行此指令*

    ```bash
    $ vagrant init
    ```

- **啟用 Vagrant**

    ```bash
    $ vagrant up
    ```

- **查看 Vagrant**

    ```bash
    $ vagrant status
    ```

- **查看 Vagrant SSH 資訊**

    ```bash
    $ vagrant ssh-config

    Host app
    HostName 127.0.0.1
    User vagrant
    Port 2202
    UserKnownHostsFile /dev/null
    StrictHostKeyChecking no
    PasswordAuthentication no
    IdentityFile /path/to-your-project/.vagrant/machines/app/virtualbox/private_key
    IdentitiesOnly yes
    LogLevel FATAL
    ```

    > 這裡的 `IdentityFile` 使用到的 SSH Key 會在 ansible 連線設定的時候要用到

- **連線進入 Vagrant**

    ```bash
    $ vagrant ssh
    ```

## Ansible 設定

#### inventory 的 local host 設定

- **建立 hosts 檔案**

	```bash
	$ cp inventroy/local/hosts.example inventroy/local/hosts
	```

- **編輯 hosts 檔案**

	```bash
	$ vim inventroy/local/hosts
	```

	```ini
	[local]
	app

	[local:vars]
	ansible_ssh_host=192.168.50.50
	ansible_ssh_port=22
	ansible_user=vagrant
	ansible_ssh_private_key_file=./.vagrant/machines/app/virtualbox/private_key
	````

	設定說明:

	| Parameters                     | Description                                                                                              |
	|:------------------------------ |:-------------------------------------------------------------------------------------------------------- |
	| `[local]`                      | ansible 的 yaml 設定檔指定的 host name                                                                   | 
	| `[local:vars]`                 | 設定 hosts 的連線資訊                                                                                    |
	| `ansible_ssh_host`             | 要操作的 host 主機 ip，這裡可以用 `Vagrantfile` 設定的 `private_network` ip                              |
	| `ansible_ssh_port`             | ansible 要用 ssh 連線進去的 port，通常都會是 `22`                                                        |
	| `ansible_user`                 | ansible ssh 連線進去的使用者名稱，如果是用 Vagrant 的話，user 要使用 `vagrant` |
	| `ansible_ssh_private_key_file` | ansible ssh 連線要使用的 private_key                                                                     |

---

##  Semaphore 的設定

> - 官方文件 - [Semaphore docker install](https://docs.ansible-semaphore.com/administration-guide/installation#docker)
> - 教學影片 - [This web UI for Ansible is so damn useful!](https://www.youtube.com/watch?v=NyOSoLn5T5U)

#### 基本設定

Semaphore 是一個 Ansible 圖形化介面的工具，可以透過 `docker compose` 來啟用

- **前置作業**

	1. 在 `docker-compose.yml` 變更 `SEMAPHORE_ADMIN_PASSWORD` 的密碼
	2. 使用 `head -c32 /dev/urandom | base64` 產生一組字串，替換掉 `docker-compose.yml` 檔案中 `SEMAPHORE_ACCESS_KEY_ENCRYPTION` 變數的值

- **啟用 Docker**

    ```bash
    $ docker-compose up -d
    ```

- 在網址輸入 http://localhost:3000

#### UI 操作

- **設定 Key store**

	- 建立 Local vagrant ssh

		把 `vagrant ssh-config` 拿到的 private_key 內容貼在裡面，然後儲存
  
	  ![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728230331.png)

   - 建立一組 None 的 key

		因為在 local repository 裏面，是不需要透過 ssh 的方式進行連線，所以可以設定 `None`
		
		![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728230601.png)
  
- **設定 Inventory**

	- 因為是使用 Docker Compose 的方式把 Semaphore 跑起來，所以我們也需要把檔案透過 `volume` 的方式同步到 docker 裏面，預設的目錄會是 `/home/semaphore`

	- 指定要使用的 inventory

		![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728230901.png)

- **設定 Repositories**

	因為是使用 Docker Compose 把服務跑起來，所以設定 repository 的路徑就用預設的 `home/semaphore` 即可

	![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728230931.png)

- **設定 Environment**

	如果有任何環境變數可以在這裡建立，但因為目前 demo 不需要，所以先寫一個 all，然後裡面是空的 `{}`

	![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728231158.png)

- **設定 Task Templates**

	- 建立 Task，就依照剛剛建立的資訊，依序選取起來，並儲存

	- 在右下角如果勾選 **Allow CLI args in Task**，就可以在輸入匡中輸入參數

	![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728231347.png)

- **執行 Ansible Playbook**

	在要執行的 Task 最左側按下 **Run** 的按鈕，進行 ansible 的部署

	![](https://raw.githubusercontent.com/bon3409/static-file/master/static-file/Pasted%20image%2020230728233450.png)