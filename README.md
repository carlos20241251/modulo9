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
practica II Desplieque de una VM con terraform en digital Ocean
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
