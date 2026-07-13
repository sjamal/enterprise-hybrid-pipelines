# Runbook: Post-Provisioning Lifecycle & Bootstrap Operations

## 1. Context Overview
This manual governs the orchestration steps required to initialize network interfaces, hostname strings, and compliance engine configuration links on virtual machine nodes following core provisioning allocation tasks.

## 2. Pre-Execution Validation
Before driving the configuration state engines, execution nodes run localized checks to evaluate resource availability and target line-of-sight network routing.

```bash
# Execute pre-flight validations inside a targeted environment tier (e.g., QA)
# Arguments: <Target_Hostname> <Staging_Context>
bash scripts/preflight_vss_wrapper.sh "t1-qa-tor-appserver-01" "QA"
```

## 3. Playbook Orchestration
Once the pre-flight checks clear, the system triggers the bootstrap playbook via the configuration engine. This ensures the handoff to configuration tools runs smoothly.

```bash
# Run the post-provisioning baseline configuration playbook using dynamic tags
ansible-playbook ansible/bootstrap_hybrid_node.yml \
  --inventory "t1-qa-tor-appserver-01.internal.institutional.edu," \
  --extra-vars "environment_tier=QA network_zone=tier1_app puppet_master_fqdn=puppetmaster.internal.institutional.edu" \
  --user="root" \
  --private-key="/opt/pipeline/secrets/id_rsa"
```

## 4. Verification Checklists
Upon playbook completion, run these non-interactive diagnostic tasks on the target instance to confirm success:

- [ ] Verify the host name maps correctly to the zone template: `hostname -f`
- [ ] Confirm the target management agent configuration exists: `cat /etc/puppetlabs/puppet/puppet.conf`
- [ ] Validate line-of-sight connectivity to the management master: `curl -k https://institutional.edu`
- [ ] Confirm background execution tools are active: `systemctl status puppet`
