# modulo9
practica I Instalacion de Webmin
sudo apt update && sudo apt upgrade -y

Agrega el repositorio oficial de Webmin:

curl -o setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/setup-repos.sh

sudo sh setup-repos.sh

Está ejecutando el script setup-repos.sh con privilegios de superusuario.

Instala Webmin:

sudo apt install webmin -y

Paso 3: Verificar que Webmin está corriendo

sudo systemctl status webmin


Abre tu navegador web e ingresa a:

https://tu_ip_del_servidor:10000

///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
practica II Despliegue de una VM con terraform en digital Ocean
instalar Terraform 

wget -4 -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform -y

Paso 2: Verificar que Terraform se instaló
terraform --version 
despues   Generar llave SSH
 ssh-keygen -t rsa -b 4096 -N ""
ENTER  ahora para ver la llave  
cat ~/.ssh/id_rsa.pub
copiar la llave publica dirigirse a digital ocean setting = security = add SSH 
poner 30 dias  y dar  full Access  generate token  copiar el token y guardar 
en una pestaña del browser 
crear archivos terrafon  usar 
mkdir ~/terraform-do
cd ~/terraform-do
crear archivo de configuracion 
cat > main.tf << 'EOF'
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

resource "digitalocean_droplet" "os3vm" {
  image    = "ubuntu-22-04-x64"
  name     = "OS3vm"
  region   = "nyc1"
  size     = "s-1vcpu-1gb"
  ssh_keys = [var.ssh_fingerprint]
}

output "ip_address" {
  value       = digitalocean_droplet.os3vm.ipv4_address
  description = "IP pública de la VM"
}
EOF
verificar que se creo  con cat main.tf
cat > variables.tf << 'EOF'
variable "do_token" {
  description = "DigitalOcean API Token"
  type        = string
  sensitive   = true
}

variable "ssh_fingerprint" {
  description = "SSH Key Fingerprint"
  type        = string
}
EOF

Verificar que se creó variables.tf

cat variables.tf

Paso 13: Crear archivo terraform.tfvars con TUS datos

nano terraform.tfvars

do_token         = "aqui va el token  que se copio"
ssh_fingerprint  = "aqui va el ssh que se  guardo"
verificar que se creo
cat terraform.tfvars
terraform init  para iniciarlo 
terraform  apply 
copiar la ip publica 
volver a la termina entrar SSH  root@ip_que se genero 
/////////////////////////////////////////////////////////////////////////////////////////////////////////////////
practica II Instalación de Ansible
# Instalar Ansible y dependencias
sudo apt install ansible python3 python3-pip sshpass -y

# Instalar pywinrm para Windows
sudo pip3 install pywinrm --break-system-package
# Verificar instalación
ansible --version
python3 -c "import winrm; print('pywinrm OK')"

#Crear Usuario Ansible en Debian
sudo adduser ansible
# Contraseña: Ansible123!
# Presiona Enter en todos los campos adicionales
sudo usermod -aG sudo ansible
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
sudo chmod 440 /etc/sudoers.d/ansible
#verificar 
id ansible
sudo cat /etc/sudoers.d/ansible
#cambiar a usuario ansible
su - ansible
# Generar llave SSH (presiona Enter en todo)
ssh-keygen -t rsa -b 4096
# Verificar
ls -la ~/.ssh/
exit
# CREAR Y CONFIGURAR DIGITALOCEAN
esta esta realizada desde la practica anterior
Configurar el Droplet
# Desde Debian, conectarse al droplet

ssh root@143.198.191.31
# Ingresar password que creaste
apt update && apt upgrade -y
elegir la primra opcion  y ok tab ok
adduser ansible
# Contraseña: Ansible123!
# Enter en todos los campos


# Configurar sudo sin contraseña

usermod -aG sudo ansible

echo "ansible ALL=(ALL) NOPASSWD:ALL" | tee /etc/sudoers.d/ansible

chmod 440 /etc/sudoers.d/ansible


# Configurar SSH - IMPORTANTE

nano /etc/ssh/sshd_config

# Configurar SSH - IMPORTANTE

nano /etc/ssh/sshd_config


# Buscar y DESCOMENTAR (quitar #) estas líneas:

# De esto:

#PubkeyAuthentication yes

#PasswordAuthentication yes


# A esto:

PubkeyAuthentication yes #debe estar descomentada 

PasswordAuthentication yes # debe estar descomentada 


# Guardar: Ctrl+O, Enter, Ctrl+X
# Reiniciar SSH

systemctl restart sshd


# Probar usuario ansible
su - ansible
sudo whoami # Debe decir "root" sin pedir contraseña

exit
exit # Salir del droplet

2.3 Copiar Llave SSH al Droplet

bash
# En Debian, como usuario ansible
su - ansible

# Copiar llave SSH
ssh-copy-id ansible@143.198.191.31
# Contraseña: Ansible123!

# Probar conexión sin contraseña
ssh ansible@143.198.191.31
# Debe conectar SIN pedir contraseña
hostname # Debe mostrar: ansible1
exit
# Volver a root

exit
# CONFIGURAR WINDOWS (ansible2)

3.1 Cambiar Red a Privada (INTERFAZ GRÁFICA)

Opción A: Configuración de Windows (Recomendado)

1. Presiona Windows + I (o busca "Configuración")

2. Click en "Red e Internet"

3. Click en "Ethernet" (o "Wi-Fi" si usas inalámbrico)
Click en tu conexión activa (ejemplo: "Red" o "Ethernet0")
4. Click en tu conexión activa (ejemplo: "Red" o "Ethernet0")
5. En "Perfil de red" selecciona "Privada"

6. Cierra la ventana
powershell

# Abrir PowerShell como Administrador

Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private


# Verificar

Get-NetConnectionProfile

# Debe mostrar: NetworkCategory : Private

3.2 Configurar Reglas de Firewall

# Abrir PowerShell como Administrador (obviar si esta como administrador

netsh advfirewall firewall add rule name="WinRM HTTP" dir=in action=allow protocol=TCP localport=5985

# Verificar

netsh advfirewall firewall show rule name="WinRM HTTP"


3.3 Configurar WinRM

powershell

# Abrir PowerShell como Administrador


# 1. Habilitar WinRM

winrm quickconfig -force


# 2. Configurar autenticación Basic en servicio

winrm set winrm/config/service/auth '@{Basic="true"}'


# 3. Configurar autenticación Basic en cliente

winrm set winrm/config/client/auth '@{Basic="true"}'


# 4. Permitir conexiones no encriptadas

winrm set winrm/config/service '@{AllowUnencrypted="true"}'


# Si da error, usar:

Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true -Force


# 5. Configurar TrustedHosts

Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force


# 6. Aumentar límite de memoria

winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
3.4 Configurar UAC para Usuarios Remotos (CRÍTICO)

powershell

# PASO MÁS IMPORTANTE

New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy" -Value 1 -PropertyType DWord -Force


# Verificar

Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "LocalAccountTokenFilterPolicy"

# Debe mostrar: LocalAccountTokenFilterPolicy : 1

3.5 Crear Usuario Ansible

powershell

# Crear usuario

net user ansible Ansible123! /add


# Agregar a Administradores

# Si tu Windows está en ESPAÑOL:

net localgroup Administradores ansible /add


# Si tu Windows está en INGLÉS:

net localgroup Administrators ansible /add


# Verificar

net user ansible

3.6 Reiniciar WinRM

powershell

# Reiniciar servicio

Restart-Service WinRM


# Verificar

Get-Service WinRM

# Debe mostrar: Status: Running

3.7 Obtener IP de Windows

powershell

# Ver IP de Windows

ipconfig


# Buscar "Dirección IPv4" (ejemplo: 10.0.0.62)

# ANOTAR ESTA IP
4.1 Crear directorio y archivo de inventario
bash
# En Debian, como root o con sudo
sudo mkdir -p /etc/ansible
sudo nano /etc/ansible/hosts

4.2 Contenido del inventario
# GRUPO: LINUX
# Cliente Linux en DigitalOcean (ansible1)
[linux]
ansible1 ansible_host=138.68.14.138 ansible_user=ansible ansible_ssh_private_key_file=/home/ansible/.ssh/id_rsa

# GRUPO: WINDOWS
# Cliente Windows (ansible2)
[win]
ansible2 ansible_host=10.0.0.159 ansible_user=ansible ansible_password=Ansible123! ansible_connection=winrm ansible_port=5985 ansible_winrm_server_cert_validation=ignore
5.1 Cambiar a usuario ansible

# En Debian
su - ansible
5.2 Probar Linux (ansible1)
ansible linux -m ping

5.3 Probar Windows (ansible2)
ansible win -m win_ping
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
practica IV Comandos Ad-Hoc
Crear archivos en el escritorio de windows 
click derecho para crear nuevo archivo txt crear otro
entrar escribir algo y guardar despues ir al explorer buscar documents para ver que estan vacia despues ejecutar siguiente commando

ansible win -m win_copy -a 'src=C:/Users/ansible/Desktop/nombre.txt dest=C:/Users/ansible/Documents/nombre.txt remote_src=yes'

donde diga nombre poner nombre del archivo ojo hacer un archivo por archivo  

ansible ansible1 -m reboot -b
antes de ejecutarlo abrir 
abrir otra terminal

su - ansible
contraseña: Ansible123!

para hacer ping a la maquina ocean digital
ansible linux -m ping
despues ejecutar el comando reboot y volver a la terminal y ejecutar el ping hasta que se pueda ejecutar 
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
Practica 5: Playbooks
ingresar  su -ansible
contrase: Ansible123!

antes de ser creado ir a windows search y buscar notepad++

volver a Debian  
nano notepad.yml
---
- name: Instalar Notepad++ en Windows
  hosts: win
  tasks:
    - name: Crear directorio temporal
      win_file:
        path: C:\temp
        state: directory
 
    - name: Descargar Notepad++
      win_get_url:
        url: https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v8.6.2/npp.8.6.2.Installer.x64.exe
        dest: C:\temp\notepad-installer.exe
 
    - name: Instalar Notepad++
      win_package:
        path: C:\temp\notepad-installer.exe
        arguments: /S
        state: present
        product_id: Notepad++
 
    - name: Verificar instalación
      win_stat:
        path: C:\Program Files\Notepad++\notepad++.exe
      register: notepad_installed
 
    - name: Mostrar resultado
      debug:
        msg: "Notepad++ instalado correctamente"
      when: notepad_installed.stat.exists
pegar crtl o crtl x 

Para que se descargue
ansible-playbook notepad.yml

ir a windows buscar notepad++ enter etc.

para el segundo 
ir a la consola digital ocean  systemctl status nginx para ver que no esta instalado
volver a la terminal servidor 
nano nginx.yml
---
- name: Instalar y configurar NGINX en Linux
  hosts: linux
  become: yes
  tasks:
    - name: Actualizar cache de paquetes
      apt:
        update_cache: yes
 
    - name: Instalar NGINX
      apt:
        name: nginx
        state: present
 
    - name: Iniciar servicio NGINX
      service:
        name: nginx
        state: started
        enabled: yes
 
    - name: Verificar que NGINX está corriendo
      service:
        name: nginx
      register: nginx_status
 
    - name: Mostrar estado de NGINX
      debug:
        msg: "NGINX está {{ nginx_status.status.ActiveState }}"
 
    - name: Mostrar URL de acceso
      debug:
        msg: "Accede a: http://{{ ansible_default_ipv4.address }}"
ctrl o ctrl x

Para que se descargue
ansible-playbook nginx.yml

Verificar NGINX en Linux:
systemctl status nginx
