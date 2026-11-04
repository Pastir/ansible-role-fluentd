# pastir.fluentd

Fluentd log collector, packaged either as:

| `fluentd_package` | Package | Repository | Config | Service |
|---|---|---|---|---|
| `fluent-package` (default) | fluent-package LTS (6.x) | `fluentd.cdn.cncf.io/lts/<major>/ubuntu/<release>` | `/etc/fluent/fluentd.conf` | `fluentd` |
| `td-agent` | td-agent v4 (EOL) | `packages.treasuredata.com/4/ubuntu/jammy` | `/etc/td-agent/td-agent.conf` | `td-agent` |

td-agent v4 was never built for Ubuntu newer than 22.04. The role installs the jammy build, which is what existing
24.04 hosts run. Use it only for hosts that still have td-agent, and move them to `fluent-package`.

## Safety

- The configuration is written, then checked with `fluentd --dry-run`, which loads every plugin and parses the
  whole config. If the check fails, the previous files are restored and the role fails.
- Configuration changes are applied with a graceful **reload** (SIGUSR2), and buffers are kept. New gems or
  plugin files need a restart, which the role does.

## Variables

| Variable | Default | Description |
|---|---|---|
| `fluentd_package` | `fluent-package` | or `td-agent` |
| `fluentd_version` | `6.0.5` | Exact package version (`4.5.2` for td-agent) |
| `fluentd_hold` / `fluentd_allow_upgrade` | `true` / `false` | |
| `fluentd_repo_enabled` / `fluentd_lts_major` / `fluentd_repo_release` | `true` / `6` / release (jammy for td-agent) | |
| `fluentd_config` | `@include <conf_dir>/conf.d/*.conf` | Main config file content |
| `fluentd_config_files` | `[]` | `[{name, content}]` in `conf.d`. `state: absent` removes a file |
| `fluentd_gems` | `[]` | `[{name, version}]` installed with `fluent-gem` |
| `fluentd_plugin_files` | `[]` | `[{name, content}]` in `<conf_dir>/plugin` |
| `fluentd_gelf_plugin` | `false` | Bundled GELF output (`@type gelf`, for Graylog) plus the `gelf` gem |
| `fluentd_gelf_gem_version` | `3.1.0` | |
| `fluentd_service_enabled` | `true` | |

## Example: Laravel logs to Graylog

```yaml
fluentd_gelf_plugin: true
fluentd_config_files:
  - name: laravel.conf
    content: |
      <source>
        @type tail
        @label @MAIN
        path /var/www/app/storage/logs/laravel-%Y-%m-%d.log
        pos_file /var/log/fluent/laravel.log.pos
        tag laravel
        <parse>
          @type json
          time_key datetime
          time_format %iso8601
        </parse>
      </source>
  - name: graylog.conf
    content: |
      <label @MAIN>
        <match **>
          @type gelf
          host graylog.example.com
          port 12201
          protocol udp
        </match>
      </label>
```

## Migrating td-agent → fluent-package

Set `fluentd_package: fluent-package` with `fluentd_allow_upgrade: true`. The package migrates `/etc/td-agent` and
the pos files. Check that the `td-agent` service is gone afterwards.

## Testing

`molecule test`:
- fluent-package on 24.04 and 26.04, td-agent on 24.04;
- an event goes through the pipeline;
- the GELF plugin loads;
- a broken config is rolled back.

## Installation

```yaml
# requirements.yml
roles:
  - name: pastir.fluentd
    src: git+git@github.com:Pastir/ansible-role-fluentd.git
    version: main  # or a tag
```

```bash
ansible-galaxy install -r requirements.yml
```

## License

MIT
