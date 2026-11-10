# Ansible Lab — Electiva 2 (ITLA)

**Autor:** Ángel González Muñoz

## Objetivo

Crear dos servidores Linux con Docker y administrarlos mediante Ansible para consultar información del sistema, crear archivos y directorios, utilizar un `loop` y ejecutar una tarea condicional.

## Requisitos

- Docker y Docker Compose instalados y funcionando.
- Ansible instalado en la computadora anfitriona.
- Colección `community.docker`, necesaria para `ansible_connection=docker`.

En Fedora, instala Ansible y la colección:

```bash
sudo dnf install -y ansible
ansible-galaxy collection install community.docker
```

> La conexión de Ansible se realiza directamente con Docker, sin SSH ni direcciones IP. Los comandos `docker` deben funcionar para el usuario que ejecuta Ansible.

## 1. Iniciar los contenedores

Desde esta carpeta:

```bash
docker compose up -d
docker ps
```

Los contenedores se llaman `server1` y `server2` y utilizan `ubuntu:24.04`.

## 2. Instalar Python 3 en cada servidor

Ubuntu no incluye necesariamente Python en su imagen mínima. Instálalo en ambos contenedores antes de usar Ansible:

```bash
docker exec server1 bash -c 'apt-get update && apt-get install -y python3'
docker exec server2 bash -c 'apt-get update && apt-get install -y python3'
```

## 3. Verificar la conexión

```bash
ansible docker -i inventory.ini -m ping
```

Ambos servidores deben responder con `SUCCESS` y `pong`.

## 4. Ejecutar el playbook

```bash
ansible-playbook -i inventory.ini --syntax-check playbook.yml
ansible-playbook -i inventory.ini playbook.yml
```

El playbook realiza estas acciones en los dos servidores:

1. Muestra el nombre del servidor y el sistema operativo.
2. Crea `/tmp/ansible-demo`.
3. Crea `/tmp/ansible-demo/info.txt` con el nombre del servidor y la distribución.
4. Crea `logs`, `backup` y `config` mediante un `loop`.
5. Muestra un mensaje únicamente si el sistema operativo es Ubuntu.

## 5. Comprobar los resultados

```bash
docker exec server1 ls -l /tmp/ansible-demo
docker exec server1 cat /tmp/ansible-demo/info.txt

docker exec server2 ls -l /tmp/ansible-demo
docker exec server2 cat /tmp/ansible-demo/info.txt
```

La ejecución correcta debe terminar con `failed=0` y `unreachable=0` para ambos servidores. Puedes ejecutar el playbook otra vez para observar que las tareas de creación no vuelven a modificar elementos que ya existen.

## Evidencia de la ejecución

La siguiente captura se agregará después de ejecutar el playbook correctamente en el equipo local. Debe mostrar el resumen final (`PLAY RECAP`) de los dos servidores.

![Ejecución exitosa del playbook en ambos servidores](capturas/ejecucion-playbook.png)

## Detener el laboratorio

```bash
docker compose down
```
