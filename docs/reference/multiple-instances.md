##  Multiple Instances [ArrX replacement]

Apps that used to be supported by the "ArrX" system which allowed the user to define a set of instances of a given app [as opposed to installing multiple instances one at a time] are being transitioned to a new generalized, invertory-driven approach.

The general idea is to move all the configuration into the `localhost.yml` along with other customizations.

Sonarr, Radarr and Lidarr support this new method as of this writing.


Define a list of all the instances of the all you want to create; if you don't want to customize them beyond that, this is all that's required.

The standard app tag will now set up all those instances.

For example, a SonarrX config might have looked like this:

```yaml
sonarrx:
  roles:
    - ""
    - bing
    - bang
    - boing
```

That would be set up in `settings.yml`.

With this new method, that will be replaced with this in `localhost.yml`:

```
sonarr_instances: ["sonarr", "sonarrbing", "sonarrbang", "sonarrboing"]
```

Note a primary changepoint is that the **entire name of the instance** is specified.  The role will no longer prepend the name of the app.

Those names have to be unique across all of your containers, so it is suggested that you keep with the `rolename+suffix` pattern for these additional instances.

## per-instance customization

Any instance defined can edit the following set of variables on a per instance basis in `localhost.yml`:

[replacing instance with the actual instance, of course, i.e. sonarr2_web_subdomain, etc.]

```
instance_web_subdomain
instance_web_domain
instance_web_port
instance_traefik_sso_middleware
instance_docker_image_repo
instance_docker_image_tag
instance_docker_ports_defaults
instance_docker_ports_ui
instance_docker_ports_custom
instance_themepark_enabled
instance_themepark_domain
instance_themepark_theme
instance_docker_envs_default
instance_docker_envs_custom
instance_docker_commands_default
instance_docker_docker_commands_custom
instance_docker_volumes_default
instance_docker_volumes_custom
instance_docker_volumes_theme
instance_docker_devices_default
instance_docker_devices_custom
instance_docker_hosts_default
instance_docker_hosts_custom
instance_docker_labels_default
instance_docker_labels_custom
instance_docker_networks_default
instance_docker_networks_custom
instance_docker_capabilities_default
instance_docker_capabilities_custom
instance_docker_security_opts_default
instance_docker_security_opts_custom
```