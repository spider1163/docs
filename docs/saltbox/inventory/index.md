# Inventory

Advanced use cases that would normally require editing roles can now be handled through the inventory system instead. 

Any variables defined in `/srv/git/saltbox/role_name/defaults/main.yml` are changeable by the user. This implementation avoids git merge conflicts when updating Saltbox.

Plex Default Variables:
``` yaml
##########################################################################
# Title:         Saltbox: Plex | Default Variables                       #
# Author(s):     desimaniac, salty                                       #
# URL:           https://github.com/saltyorg/Saltbox                     #
# --                                                                     #
##########################################################################
#                   GNU General Public License v3.0                      #
##########################################################################
---
################################
# Basics
################################

plex_name: plex

################################
# Paths
################################

plex_paths_folder: "{{ plex_name }}"
plex_paths_location: "{{ server_appdata_path }}/{{ plex_paths_folder }}"
plex_paths_transcodes_location: "{{ transcodes_path }}/{{ plex_paths_folder }}"
plex_paths_folders_list:
  - "{{ plex_paths_location }}"
  - "{{ plex_paths_location }}/Library"
  - "{{ plex_paths_location }}/Library/Application Support"
  - "{{ plex_paths_location }}/Library/Application Support/Plex Media Server"
  - "{{ plex_paths_location }}/Library/Application Support/Plex Media Server/Plug-ins"
  - "{{ plex_paths_location }}/Library/Logs"
  - "{{ plex_paths_location }}/Library/Logs/Plex Media Server"
  - "{{ plex_paths_transcodes_location }}"
plex_paths_application_support_location: "{{ plex_paths_location }}/Library/Application Support/Plex Media Server"
plex_paths_config_location: "{{ plex_paths_application_support_location }}/Preferences.xml"
plex_paths_log_location: "{{ plex_paths_application_support_location }}/Logs"
plex_paths_plugins_location: "{{ plex_paths_application_support_location }}/Plug-ins"
plex_paths_plugin_support_location: "{{ plex_paths_application_support_location }}/Plug-in Support"
plex_paths_db_location: "{{ plex_paths_plugin_support_location }}/Databases/com.plexapp.plugins.library.db"

################################
# Web
################################

plex_web_subdomain: "{{ plex_name }}"
plex_web_domain: "{{ user.domain }}"
plex_web_port: "32400"
plex_web_url: "{{ 'https://' + plex_web_subdomain + dns_record_suffix + '.' + plex_web_domain
               if (reverse_proxy_is_enabled)
               else 'http://localhost:' + plex_web_port }}"

################################
# DNS
################################

plex_dns_record: "{{ plex_web_subdomain }}"
plex_dns_zone: "{{ plex_web_domain }}"

################################
# Traefik
################################

plex_traefik_middleware: "secureHeaders@file"
plex_traefik_certresolver: "{{ traefik_default_certresolver }}"
plex_traefik_enabled: true

################################
# THEME
################################

# Options can be found at https://github.com/gilbN/theme.park
plex_themepark_enabled: false
plex_themepark_theme: "{{ global_themepark_theme }}"

################################
# Settings
################################

plex_open_main_ports: false
plex_db_cache_size: default

################################
# Docker
################################

# Container
plex_docker_container: "{{ plex_name }}"

# Image
plex_docker_image_pull: true
plex_docker_image_tag: "public"
plex_docker_image: "plexinc/pms-docker:{{ plex_docker_image_tag }}"

# Ports
plex_docker_ports_defaults:
  - "{{ plex_web_port }}-{{ plex_web_port | int + 10 }}:{{ plex_web_port }}"
plex_docker_ports_custom: []
plex_docker_ports_webtools:
  - "33400-33410:33400/tcp"
  - "33443-33453:33443/tcp"
plex_docker_ports: "{{ plex_docker_ports_defaults
                        + plex_docker_ports_webtools
                        + plex_docker_ports_custom
                    if (not reverse_proxy_is_enabled
                        or plex_open_main_ports)
                    else plex_docker_ports_webtools
                        + plex_docker_ports_custom }}"
# Envs
plex_docker_envs_advertise_ip: "{{ plex_web_url + ':443'
                                if (reverse_proxy_is_enabled)
                                else '' }}"
plex_docker_envs_default:
  PLEX_UID: "{{ uid }}"
  PLEX_GID: "{{ gid }}"
  PLEX_CLAIM: "{{ (plex_claim_code) | default(omit) }}"
  CHANGE_CONFIG_DIR_OWNERSHIP: "false"
  NVIDIA_DRIVER_CAPABILITIES: "{{ 'compute,video,utility' if (gpu.nvidia) | default(false) else omit }}"
  NVIDIA_VISIBLE_DEVICES: "{{ 'all' if (gpu.nvidia) | default(false) else omit }}"
  TZ: "{{ tz }}"
  ADVERTISE_IP: "{{ plex_docker_envs_advertise_ip }}"
plex_docker_envs_custom: {}
plex_docker_envs: "{{ plex_docker_envs_default
                          | combine(plex_docker_envs_custom) }}"

# Commands
plex_docker_commands_default: []
plex_docker_commands_custom: []
plex_docker_commands: "{{ plex_docker_commands_default
                          + plex_docker_commands_custom}}"

# Volumes
plex_docker_volumes_default:
  - "{{ plex_paths_location }}:/config"
  - "{{ server_appdata_path }}/scripts:/scripts"
  - "/mnt/unionfs/Media:/data"
  - "/mnt:/mnt"
  - "/tmp:/tmp"
  - "/dev/shm:/dev/shm"
  - "{{ plex_paths_transcodes_location }}:/transcode"
plex_docker_volumes_custom: []
plex_docker_volumes_theme:
  - "{{ plex_paths_location }}/98-themepark.sh:/etc/cont-init.d/98-themepark"
plex_docker_volumes: "{{ plex_docker_volumes_default
                          + plex_docker_volumes_custom
                          + docker_volumes_downloads_common
                      if not (plex_themepark_enabled)
                      else plex_docker_volumes_default
                          + plex_docker_volumes_custom
                          + docker_volumes_downloads_common
                          + plex_docker_volumes_theme}}"

# Devices
plex_docker_devices_default: []
plex_docker_devices_custom: []
plex_docker_devices: "{{ plex_docker_devices_default
                          + plex_docker_devices_custom }}"

# Hosts
lazyman_ip: "{{ ( lookup('dig', 'nhl.freegamez.ga', '@8.8.8.8', 'qtype=A') | ipv4 ) | default(false,true) }}"

plex_docker_hosts_default:
  "metric.plex.tv": "{{ ip_address_localhost }}"
  "metrics.plex.tv": "{{ ip_address_localhost }}"
  "analytics.plex.tv": "{{ ip_address_localhost }}"
  "mf.svc.nhl.com": "{{ lazyman_ip | ternary(lazyman_ip, omit) }}"
  "mlb-ws-mf.media.mlb.com": "{{ lazyman_ip | ternary(lazyman_ip, omit) }}"
  "playback.svcs.mlb.com": "{{ lazyman_ip | ternary(lazyman_ip, omit) }}"

plex_docker_hosts_custom: []
plex_docker_hosts: "{{ docker_hosts_common
                        | combine(plex_docker_hosts_default)
                        | combine(plex_docker_hosts_custom) }}"

# Labels
plex_docker_labels_default: {}
plex_docker_labels_custom: {}
plex_docker_labels: "{{ docker_labels_common
                        | combine(plex_docker_labels_default)
                        | combine(plex_docker_labels_custom) }}"

# Hostname
plex_docker_hostname: "{{ plex_name }}"

# Networks
plex_docker_networks_alias: "{{ plex_name }}"
plex_docker_networks_default: []
plex_docker_networks_custom: []
plex_docker_networks: "{{ docker_networks_common
                          + plex_docker_networks_default
                          + plex_docker_networks_custom }}"

# Capabilities
plex_docker_capabilities_default: []
plex_docker_capabilities_custom: []
plex_docker_capabilities: "{{ plex_docker_capabilities_default
                              + plex_docker_capabilities_custom }}"

# Security Opts
plex_docker_security_opts_default: []
plex_docker_security_opts_custom: []
plex_docker_security_opts: "{{ plex_docker_security_opts_default
                                + plex_docker_security_opts_custom }}"

# Restart Policy
plex_docker_restart_policy: unless-stopped

# State
plex_docker_state: started
```

Should you require additional functionality then by all means create an issue on the [main repository](https://github.com/saltyorg/Saltbox/) and we'll look at accomodating it.

Overrides are done in this file `"/srv/git/saltbox/inventories/host_vars/localhost.yml"` which does not exist by default.

Changing Sonarr/Radarr image tags:
``` yaml
radarr_docker_image_tag: nightly
sonarr_docker_image_tag: nightly
```
Which would override the docker image tags used when deploying the containers.