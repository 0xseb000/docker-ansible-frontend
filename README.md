# docker-ansible-frontend

## Projektstruktur

Das Projekt besteht aus einer Ansible-Control-Maschine und zwei Zielsystemen.
Die Ansible-Maschine verwaltet die beiden Targets per SSH und installiert dort automatisiert Docker, Docker Compose und eine Webanwendung mit Nginx.

### Infrastruktur

| VM    | Hostname   | Rolle                | IP-Adresse     |
| ----- | ---------- | -------------------- | -------------- |
| VM105 | `ansible`  | Ansible-Control-Node | `10.10.10.105` |
| VM106 | `target01` | Docker-Webserver 1   | `10.10.10.106` |
| VM107 | `target02` | Docker-Webserver 2   | `10.10.10.107` |

Alle Maschinen befinden sich im internen Netzwerk `10.10.10.0/24`.
Der Proxmox-Host stellt über `vmbr5` das Gateway `10.10.10.1` bereit.

### Architektur

```text
+-----------------------------+
| VM105                       |
| Hostname: ansible           |
| Rolle: Ansible-Control-Node |
| IP: 10.10.10.105            |
+--------------+--------------+
               |
               | SSH / Ansible
               |
       +-------+-------+
       |               |
       v               v
+-------------+   +-------------+
| VM106       |   | VM107       |
| target01    |   | target02    |
| 10.10.10.106|   | 10.10.10.107|
+------+------+   +------+------+
       |                 |
       | Docker          | Docker
       | Docker Compose  | Docker Compose
       | Nginx-Frontend  | Nginx-Frontend
       v                 v
http://10.10.10.106   http://10.10.10.107
```

### Ansible-Projektstruktur

```text
ansible-project/
├── ansible.cfg
├── inventory/
│   └── hosts.yml
├── group_vars/
│   └── webservers.yml
├── playbooks/
│   └── site.yml
└── roles/
    ├── common/
    │   ├── tasks/
    │   │   └── main.yml
    │   └── handlers/
    │       └── main.yml
    ├── docker/
    │   └── tasks/
    │       └── main.yml
    └── webapp/
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   ├── docker-compose.yml.j2
        │   ├── nginx.conf.j2
        │   └── index.html.j2
        └── handlers/
            └── main.yml
```

### Beschreibung der wichtigsten Dateien

| Datei / Ordner              | Beschreibung                                                                     |
| --------------------------- | -------------------------------------------------------------------------------- |
| `ansible.cfg`               | Enthält die Grundeinstellungen für Ansible, zum Beispiel den Pfad zum Inventory. |
| `inventory/hosts.yml`       | Enthält die Zielsysteme `target01` und `target02`.                               |
| `group_vars/webservers.yml` | Enthält Variablen, die für beide Webserver gelten.                               |
| `playbooks/site.yml`        | Haupt-Playbook, das alle Rollen ausführt.                                        |
| `roles/common/`             | Allgemeine Vorbereitung der Zielsysteme, zum Beispiel Paketupdates.              |
| `roles/docker/`             | Installiert Docker und Docker Compose auf den Targets.                           |
| `roles/webapp/`             | Erstellt und startet die Webanwendung mit Nginx im Docker-Container.             |

### Ziel des Deployments

Nach dem Ausführen des Ansible-Playbooks sollen beide Zielsysteme automatisch eingerichtet sein.

Auf beiden Targets wird folgendes umgesetzt:

* Docker wird installiert.
* Docker Compose wird eingerichtet.
* Eine Nginx-Webanwendung wird als Container gestartet.
* Eine einfache HTTP-Seite wird bereitgestellt.
* Die Website ist im internen Netzwerk erreichbar.

### Aufruf der Webanwendung

Nach erfolgreichem Deployment können die Webseiten über folgende Adressen getestet werden:

```text
http://10.10.10.106
http://10.10.10.107
```

Alternativ kann der Test von der Ansible-Maschine aus mit `curl` durchgeführt werden:

```bash
curl http://10.10.10.106
curl http://10.10.10.107
```

### Zielbild

Das Ziel ist, dass die VM105 als zentrale Verwaltungsmaschine dient.
Von dort aus werden die beiden Zielsysteme VM106 und VM107 automatisiert konfiguriert. Dadurch wird gezeigt, wie mehrere Linux-Server mit Ansible verwaltet und Docker-Anwendungen automatisiert bereitgestellt werden können.
