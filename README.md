# Ansible Execution Environment for Homelab

- Use `quay.io/centos/centos:stream9-minimal` as the base image
- Use Python `3.11` as Python interpreter
- Use Ansible `2.15.*` and Ansible Runner `2.3.*` to run playbooks on EE
- Add RPM Packages that listed in [`dependencies/bindep.txt`](dependencies/bindep.txt) for basic remote connection and debugging
- Add Python packages that listed in [`dependencies/requirements.txt`](ependencies/requirements.txt)
- Add Ansible collections that listed in [`dependencies/requirements.yml`](dependencies/requirements.yml)
- Customize configuration for `ansible-galaxy` (`additional_build_files` and `additional_build_steps`)
  - Add customized `ansible.cfg` to the context (`additional_build_files`)
  - Place customized file as `~/.ansible.cfg` for the stage that `ansible-galaxy` is invoked (`prepend_galaxy` under `additional_build_steps`)
- Run additional commands during build steps (`additional_build_steps`)
  - To allow the hard-coded interpreter name (`python3`) passed by AWX, `alternatives` command is appended under `append_base` to make the binary `/usr/bin/python3.11` executable as command `python3`


On a new release, docker image will be built and published
