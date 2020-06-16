# TestProjectOne

### 环境
- 用terraform创建了三台Azure CentOS 7.5虚机：一台跳板机（有公网IP）、一台master01（无公网IP）、一个data01（无公网IP）。
- 然后在跳板机上安装ansible。
- tools_install_settings暂时依赖Centos 7.5 local repository file。需要手动下载这两个文件：azure-cli.repo、kubernetes.repo。

### 脚本
- stop_firewalld_selinux_roles.yml执行的任务有：
  +  ```
     $ systemctl stop firewalld
     $ systemctl disable firewalld
     ```
  + $ setenforce 0
  + $ vi /etc/selinux/config
- install_disk_roles.yml执行的任务有：
  + mkfs -t ext4 /dev/sdc
  + mkdir /data
  + sudo mount /dev/sdc /data（没用mount -a，因为没成功找到/dev/sdc的UUID）
- basic_linux_settings_roles.yml执行的任务有：
  + disable firewalld
  + disable selinux
  + disable ipv6
  + set timezone to Asia/Tokyo
- linux_user_settings_roles.yml执行的任务有：
  + create group "infra-user001" with group id 1000
  + create user "infra-user001" with group id 1000
- tools_install_settings_roles.yml执行的任务有：
  + install az-cli
  + install kubectl


### 执行命令
- 安装ansible后，
  + cd /etc/ansible/roles
  + 若不想把每个任务节点的host key加入~/.ssh/known_hosts文件，可选：export ANSIBLE_HOST_KEY_CHECKING=False
  + ansible-playbook -C 选项不实际执行改变，会检查语法和预测发生的变化。去掉-C就可以实际运行了。
  + ansible-playbook -C stop_firewalld_selinux_roles.yml --extra-vars 'ansible_sudo_pass=P@ssw0rd1234!'
  + ansible-playbook -C install_disk_roles.yml --extra-vars 'ansible_sudo_pass=P@ssw0rd1234!'
  + ansible-playbook -C basic_linux_settings_roles.yml --extra-vars 'ansible_sudo_pass=P@ssw0rd1234!'
  + ansible-playbook -C linux_user_settings_roles.yml --extra-vars 'ansible_sudo_pass=P@ssw0rd1234!'
  + ansible-playbook -C tools_install_settings_roles.yml --extra-vars 'ansible_sudo_pass=P@ssw0rd1234!'
    * 因为目标机器没有对应repository file，检查时会报错
      ```
      {"changed": false, "msg": "No package matching 'kubectl' found available, installed or updated", "rc": 126, "results": ["No package matching 'kubectl' found available, installed or updated"]}
      ```
      这个不影响实际执行，实际执行时会把repository file拷贝到目标机器/etc/yum.repos.d文件夹。
- 检查执行结果。在本地试验过，基本可用。脚本有待优化。
