# Ansible Role: Ollama

An Ansible role that downloads and installs [Ollama](https://ollama.com) on Debian/Ubuntu systems.

## Requirements

- Ansible 2.16 or later
- Target hosts running Debian or Ubuntu

## Role Variables

Available variables are listed below, along with some sample values. All have default values, `defaults/main.yaml` for a full list of these.

```yaml
ollama_ver: 0.19.0
ollama_download_base_url: https://github.com/ollama/ollama/releases/download
```

Use `ollama_ver` specify the ollama version to use, or use `latest` to always install the most recent version when running this role. It should normally not be necessary to change the base download URL.

```yaml
ollama_path_prefix: /usr/local
ollama_user_home: /var/lib/ollama
ollama_models_dir: /var/lib/ollama/models
ollama_user: ollama
ollama_group: ollama
```

Use these to manage where ollama will be installed, as well as user ollama will run under.

```yaml
ollama_host: 127.0.0.1:11434
```

Address and port for the Ollama service to listen on. Set to `0.0.0.0:11434` to allow access from the network. Note that ollama itself has no authentication – it will be exposed to anyone with access to the port, potentially also anyone on the Internet.

```yaml
ollama_origins:
  - https://example.org/myapp
```

List of allowed CORS origins for the Ollama API. When not set, Ollama's built-in defaults are used.

```yaml
ollama_models:
  - name: llama3.2
    state: present
  - name: mistral
    state: absent
  - name: some-model-from-untrusted-registry
    state: present
    insecure: true
```

List of models to pull (install) (when state is `present`) or remove (when state is `absent`).

```yaml
ollama_flash_attention: true
ollama_kv_cache_type: q4_0
ollama_context_length: 4096
```

Various model tuning values.

```yaml
ollama_debug: 2
ollama_skip_download: true
```

`ollama_debug` is the value for the `OLLAMA_DEBUG` environment variable in the service unit. Leave empty to disable debug logging. Set to `1` for standard debug output, `2` for more verbose output, etc.

`ollama_skip_download` is mainly used when testing this Ansible role to avoid downloading ollama every time the tests are run.


```yaml
ollama_uninstall: true
```

If you no longer believe in the AI hype, you can use this to uninstall ollama. When uninstalling, the following will be deleted:

* The ollama binary will be deleted (for instance `/usr/local/bin/ollama`)
* The ollama lib-directory (for instance `/usr/local/lib/ollama`) 
* The ollama user (including home directory) and group

If the model directory is outside the ollama home directory, then it will currently not be deleted.

## Upgrading ollama

The role automatically handles upgrading (or downgrading) Ollama. When the role runs and Ollama is already installed, it checks the installed version using `ollama --version`. If the installed version differs from `ollama_ver`, the role stops the service, removes only the binary (`/usr/local/bin/ollama`) and the lib directory (`/usr/local/lib/ollama`), and then installs the new version. All data, models, logs, and settings stored in `ollama_user_home` are preserved.

To always keep Ollama up to date with the latest release, set `ollama_ver` to `"latest"`:

## Service Management

The role installs a systemd service unit at `/etc/systemd/system/ollama.service` and enables and starts the `ollama` service. The service runs `ollama serve` as the `ollama` user and group, and is configured to restart automatically on failure.

## Example playbooks

```yaml
- hosts: all
  become: true
  roles:
    - role: eirikns.ollama
      vars:
        ollama_ver: latest
        ollama_models:
          - name: llama3.2
            state: present
          - name: mistral
            state: absent
          - name: some-model-from-untrusted-registry
            state: present
            insecure: true
```

To uninstall Ollama:

```yaml
- hosts: all
  become: true
  roles:
    - role: eirikns.ollama
      vars:
        ollama_uninstall: true
```

## Development

Most molecule scenarios use `ollama_skip_download: true` to avoid downloading the Ollama archive on every test run. The `download` scenario is an exception — it performs a real download and verifies the SHA256 checksum end-to-end. Because the Ollama archive is ~1.5 GB, this scenario is not included in CI.

To run it locally:

```sh
molecule test --scenario-name download
```

# License

MIT No Attribution

# Author

This Ansible role was created in 2025 by Eirik Nicolai Synnes and published as an Open Source project in 2026.

# Acknowledgements

This role uses Docker images created by Jeff Geerling for testing. The approach to developing this role, as well as how to use GitHub Actions to manage it, is inspired by his work. You can find him on GitHub at https://github.com/geerlingguy.