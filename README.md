# cac-aap-config
Configuration as Code for AAP

Ejecución en Linux:
`ansible-navigator run dispatch_config.yml \
-e "@vars.yml" \
-e "@vars_secrets.yml" \
--ask-vault-pass \
-m stdout`

Ejecución en Mac OS:
`ansible-playbook dispatch_config.yml -e "@vars_secrets.yml" --ask-vault-pass`

Nota, si da errores de Python:
`python3 -m ansible.cli.playbook dispatch_config.yml \
  -e "@vars.yml" \
  -e "@vars_secrets.yml" \
  --ask-vault-pass
`
