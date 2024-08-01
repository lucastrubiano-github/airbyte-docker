# Comandos para levantar Airbyte Docker en EC2 de AWS

Explicación paso a paso para levantar Airbyte corriendo en un contenedor Docker en una instancia EC2 de AWS.

## 1. Crear instancia EC2 en AWS

1. Ingresar a la consola de AWS.
2. Ir a la sección de EC2.
3. Hacer clic en el botón "Launch Instance".
4. Seleccionar la imagen de Amazon Linux 2.
5. Seleccionar el tipo de instancia deseado.
    Instancia: t4g.medium (2 vCPU, 4 GiB RAM)
6. Hacer clic en el botón "Next: Configure Instance Details".
7. Dejar los valores por defecto y hacer clic en el botón "Next: Add Storage".
8. Dejar los valores por defecto y hacer clic en el botón "Next: Add Tags".
9. Hacer clic en el botón "Add Tag". Si se desea, agregar un tag con la siguiente información:
    Key: Name
    Value: airbyte-conto
10. Hacer clic en el botón "Next: Configure Security Group".
11. Crear un nuevo grupo de seguridad con las siguientes reglas:
    - Type: SSH, Protocol: TCP, Port Range: 22, Source: Anywhere
    - Type: HTTP, Protocol: TCP, Port Range: 80, Source: Anywhere
    - Type: Custom TCP, Protocol: TCP, Port Range: 8000, Source: Anywhere
12. Hacer clic en el botón "Review and Launch".
13. Hacer clic en el botón "Launch".
14. Seleccionar un par de claves existente o crear uno nuevo.
15. Hacer clic en el botón "Launch Instances".
16. Hacer clic en el botón "View Instances".
17. Esperar a que la instancia esté en estado "running".
18. Hacer clic en el botón "Connect" para obtener las instrucciones de conexión a la instancia.

## 2. Conectarse a la instancia EC2

```bash
# Conectar a la instancia EC2
ssh -i "airbyte-conto.pem"
```

## 3. Instalar Docker en la instancia EC2

```bash
# Actualizar el sistema
sudo yum update -y

# Upgrade the installed packages
sudo yum upgrade -y

# Instalar Docker y Docker Compose
sudo yum install -y docker

# Iniciar el servicio de Docker
sudo service docker start

# Agregar el usuario al grupo de Docker
sudo usermod -a -G docker $USER

# Reiniciar la instancia
sudo reboot
```

```bash
# Conectar a la instancia EC2
ssh -i "airbyte-conto.pem"

# Verificar la instalación de Docker
docker --version

# Iniciar el servicio de Docker al iniciar la instancia
sudo chkconfig docker on
```

## 4. Instalar Docker Compose en la instancia EC2

```bash
# Descargar la versión estable de Docker Compose
sudo yum install -y docker-compose-plugin

# Verificar la instalación de Docker Compose
docker-compose --version
```

## 5. Copiar el archivo docker-compose.yml

```bash
# Crear un directorio para el archivo docker-compose.yml
mkdir airbyte-conto

# Ingresar al directorio
cd airbyte-conto

# Copiar el archivo docker-compose.yml a la instancia EC2
scp -i "airbyte-conto.pem" docker-compose.yml

# Verificar que el archivo docker-compose.yml se haya copiado correctamente
ls
```

## 6. Configurar el archivo .env

```bash
# Copiar el archivo .env a la instancia EC2
scp -i "airbyte-conto.pem" .env

# Verificar que el archivo .env se haya copiado correctamente
ls
```

## 7. Levantar Airbyte Docker

```bash
# Levantar Airbyte Docker
docker-compose up -d
```

## 8. Configurar Airbyte, conectores y conexiones

```bash
# Crear tunel SSH
SSH_KEY=~/Downloads/YO/YouTube/airbyte-key.pem
INSTANCE_IP=44.201.45.46
ssh -i $SSH_KEY -L 8000:localhost:8000 -N -f ec2-user@$INSTANCE_IP
ssh -i $SSH_KEY -L 8001:localhost:8001 -N -f ec2-user@$INSTANCE_IP

# Abrir en el navegador
http://localhost:8000
```

## 9. Configurar Airbyte para que se ejecute en el puerto 80

```bash
# Detener Airbyte Docker
docker-compose down

# Editar el archivo docker-compose.yml
nano docker-compose.yml

# Cambiar el puerto 8000 por el puerto 80
ports:
  - "80:8000"

# Guardar los cambios y salir del editor
Ctrl + O
Enter
Ctrl + X

# Levantar Airbyte Docker
docker-compose up -d
```

## 10. Configurar Airbyte para que se ejecute automáticamente al iniciar la instancia EC2

```bash
# Crear un archivo de servicio para Airbyte
sudo nano /etc/systemd/system/airbyte.service

# Pegar el siguiente contenido en el archivo de servicio
[Unit]
Description=Airbyte Service
After=docker.service
Requires=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/docker-compose -f /home/ec2-user/airbyte-conto/docker-compose.yml up -d
ExecStop=/usr/bin/docker-compose -f /home/ec2-user/airbyte-conto/docker-compose.yml down
WorkingDirectory=/home/ec2-user/airbyte-conto

[Install]
WantedBy=multi-user.target

# Guardar los cambios y salir del editor
Ctrl + O
Enter
Ctrl + X

# Recargar los servicios de systemd
sudo systemctl daemon-reload

# Habilitar el servicio de Airbyte
sudo systemctl enable airbyte.service

# Iniciar el servicio de Airbyte
sudo systemctl start airbyte.service

# Verificar el estado del servicio de Airbyte
sudo systemctl status airbyte.service
```

## 11. Acceder a Airbyte desde el navegador con VPN

```bash
# Acceder a Airbyte desde el navegador
http://<public-ip>
```

## 12. Detener Airbyte Docker

```bash
# Detener Airbyte Docker
docker-compose down
```