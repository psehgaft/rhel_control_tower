# RHEL Control Tower

Colección de playbooks y roles Ansible para preparar servidores RHEL 7, 8, 9 y 10 con componentes seleccionables desde el inventario.

El playbook principal habilita cada componente con una bandera `section_*` en `inventory/group_vars/all.yml`. Incluye roles para suscripción RHSM, repositorios abiertos permitidos, Identity Management (IdM/FreeIPA), usuarios/grupos y DNS, colección Ansible para IBM AIX, NFS, Podman, Registry v2, HAProxy, Keepalived, herramientas y OpenSCAP.

> **Compatibilidad:** ofrece una estructura común para RHEL 7–10. La disponibilidad de paquetes y el soporte dependen de la versión menor, arquitectura, suscripción y repositorios. RHEL 7 está fuera de mantenimiento general; valida el soporte vigente antes de usarlo en producción. Instala IdM en un host limpio y dedicado.

## Requisitos y ejecución

- Ansible en el nodo de control; SSH, Python y privilegios `become` en los nodos.
- RHSM activa si se usa contenido de Red Hat.
- Instala colecciones: `ansible-galaxy collection install -r requirements.yml`.
- Edita `inventory/hosts.yml` y `inventory/group_vars/all.yml`; reemplaza CHANGEME y activa solo las banderas necesarias.
- Cifra contraseñas con Ansible Vault. Luego valida: `ansible-playbook -i inventory/hosts.yml site.yml --syntax-check`.
- Ejecuta primero en laboratorio: `ansible-playbook -i inventory/hosts.yml site.yml --limit rhel_servers`.

## Repositorios

El rol `open_repositories` habilita solo las entradas declaradas en `open_repositories`. Agrega repositorios públicos aprobados y específicos para cada versión, con URL y llave GPG. Los repositorios oficiales se habilitan por RHSM mediante `rhsm_repositories`; no se activa indiscriminadamente cada repositorio.

## IdM

El servidor requiere FQDN estático, DNS/NTP y puertos preparados. Las zonas se declaran en `idm_dns_zones`; los registros A, como pares `name`/`ip`, en `idm_dns_records`. El usuario integrado `admin` usa el secreto de IdM. `root` es una cuenta local y no se crea como usuario de dominio. `sysadmin` y `lector` son identidades de dominio asignables a grupos; proporciona sus contraseñas mediante Vault si gestionas contraseñas con el playbook.

## Seguridad y roles adicionales

Los valores CHANGEME son marcadores, no secretos válidos. Usa una variable cifrada para RHSM, IdM, Keepalived y cualquier credencial. El registry viene con TLS habilitado; aporta certificados válidos y almacenamiento persistente. OpenSCAP solo evalúa (no remedia automáticamente); configura un perfil válido. Los binarios externos necesitan URL y checksum SHA-256.

Para agregar roles, crea `roles/<nombre>/tasks/main.yml`, agrega `section_<nombre>` al inventario y un bloque condicional `import_role` en `site.yml`.

```text
inventory/  roles/  site.yml  requirements.yml  ansible.cfg
```
