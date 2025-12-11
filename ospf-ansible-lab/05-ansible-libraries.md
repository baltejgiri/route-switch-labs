# Ansible Libraries and Modules

This section covers the Ansible collections, modules, and libraries used throughout our OSPF automation lab. It is important to understand these components for creating and modifying playbooks for network automation.

## Ansible Collections

Ansible Collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins. Think of a way to provide item in a package, here item would be playbook, role, modules or plugin and package would be collections itself.

### cisco.ios Collection

The primary collection used in this lab is **cisco.ios**, which provides modules specifically designed for managing Cisco IOS and IOS-XE network devices.

**Installation:**
```bash
ansible-galaxy collection install cisco.ios
```

**Verification:**
```bash
ansible-galaxy collection list | grep cisco.ios
```

Expected output:
```
cisco.ios                     5.3.0
```

**Documentation:**
```bash
ansible-doc -l cisco.ios
```

## Modules Used in This Lab

### 1. cisco.ios.ios_config

**Purpose:** Manages configuration sections of Cisco IOS devices using block or line-based configuration commands.

**Usage in Lab:**
This module is used in both playbooks to configure:
- OSPF router process and router ID
- OSPF network statements
- EIGRP process and network statements

**Example from OSPF playbook:**
```yaml
- name: Configure OSPF Process and Router ID
  cisco.ios.ios_config:
    lines:
      - router-id {{ router_id }}
    parents: router ospf {{ ospf_process_id }}
    save_when: modified
  register: ospf_config
```

**Example from EIGRP playbook:**
```yaml
- name: Configure EIGRP Process
  cisco.ios.ios_config:
    lines:
      - network {{ item.network }} {{ item.wildcard }}
    parents: router eigrp {{ eigrp_as }}
    save_when: modified
  loop: "{{ eigrp_networks[inventory_hostname] }}"
```

**Key Parameters Used:**

- **lines**: List of configuration commands to execute
- **parents**: Parent commands that uniquely identify the section (e.g., `router ospf 1`)
- **save_when**: When to save the running configuration
  - `modified`: Save only when configuration changes (used in the lab)
  - `always`: Save after every task execution

**How it Works:**
1. Ansible connects to the device via SSH
2. Enters configuration mode
3. Navigates to the parent configuration section
4. Applies the lines specified
5. Saves configuration if `save_when` condition is met

**Documentation:**
```bash
ansible-doc cisco.ios.ios_config
```

### 2. cisco.ios.ios_command

**Purpose:** Executes show commands on Cisco IOS devices and returns the output. This module is used for verification and gathering information, not for configuration changes.

**Usage in Lab:**
Used for all verification tasks in both playbooks:
- `show ip ospf` - OSPF process information
- `show ip ospf neighbor` - OSPF neighbor relationships
- `show ip ospf interface brief` - OSPF-enabled interfaces
- `show ip route ospf` - OSPF routes in routing table
- `show ip eigrp neighbors` - EIGRP neighbor relationships
- `show ip eigrp topology` - EIGRP topology table
- `show ip route eigrp` - EIGRP routes in routing table

**Example from OSPF playbook:**
```yaml
- name: Verify OSPF Neighbors
  cisco.ios.ios_command:
    commands:
      - show ip ospf neighbor
  register: ospf_neighbors

- name: Verify OSPF Routes
  cisco.ios.ios_command:
    commands:
      - show ip route ospf
  register: ospf_routes
```

**Example from EIGRP playbook:**
```yaml
- name: Verify EIGRP Neighbors
  cisco.ios.ios_command:
    commands:
      - show ip eigrp neighbors
  register: eigrp_neighbors
```

**Key Parameters Used:**

- **commands**: List of show commands to execute (always a list, even for single command)

**Important Notes:**
- This module does NOT make configuration changes
- Output is returned in the `stdout` and `stdout_lines` attributes
- Always use with `register` to capture output for display

**Documentation:**
```bash
ansible-doc cisco.ios.ios_command
```

## Ansible Core Keywords

### register

**Purpose:** Stores the output of a task in a variable for later use in the playbook.

**Usage in Lab:**
Every verification task uses `register` to store command outputs:

```yaml
- name: Verify OSPF Process
  cisco.ios.ios_command:
    commands:
      - show ip ospf
  register: ospf_process
```

**What Gets Registered:**
The `ospf_process` variable contains:
- `changed`: Boolean (always `false` for show commands)
- `stdout`: Command output as a list
- `stdout_lines`: Command output split into lines (list of lists)
- `failed`: Boolean indicating if command failed

**Accessing Registered Data:**
```yaml
# Display first command output
- debug:
    msg: "{{ ospf_process.stdout_lines[0] }}"

# Check if configuration changed
- debug:
    msg: "Router ID configured"
  when: ospf_config.changed
```

### debug

**Purpose:** Prints statements during playbook execution for displaying information and troubleshooting.

**Usage in Lab:**
Used extensively for displaying configuration status and verification results.

**Example - Display Configuration Details:**
```yaml
- name: Display Configuration Details
  debug:
    msg: 
      - "============================================"
      - "Configuring: {{ inventory_hostname }}"
      - "Router-ID: {{ router_id }}"
      - "OSPF Process: {{ ospf_process_id }}"
      - "Description: {{ description }}"
      - "============================================"
```

**Example - Display Verification Results:**
```yaml
- name: Display OSPF Verification for {{ inventory_hostname }}
  debug:
    msg: 
      - "============================================"
      - "{{ inventory_hostname }} - OSPF Verification"
      - "============================================"
      - ""
      - "--- OSPF Neighbors ---"
      - "{{ ospf_neighbors.stdout_lines[0] }}"
```

**Example - Conditional Display:**
```yaml
- name: Display Network Statements Configured
  debug:
    msg: "Network {{ item.item.network }}/{{ item.item.wildcard }} added to Area {{ item.item.area }}"
  loop: "{{ network_config.results }}"
  when: 
    - network_config.results is defined
    - item.changed
```

**Key Uses in Lab:**
- Display device being configured
- Show configuration changes made
- Display verification command outputs
- Show summary information

### pause

**Purpose:** Pauses playbook execution for a specified time or until user input.

**Usage in Lab:**
Used to allow time for routing protocol convergence:

**OSPF Playbook:**
```yaml
- name: Wait for OSPF Convergence
  pause:
    seconds: 20
    prompt: "Waiting for OSPF to converge across all areas..."
  run_once: true
```

**EIGRP Playbook:**
```yaml
- name: Wait for EIGRP Convergence
  pause:
    seconds: 10
    prompt: "Waiting for EIGRP to converge..."
  run_once: true
```

**Parameters Used:**
- `seconds`: Time to pause (20 seconds for OSPF, 10 seconds for EIGRP)
- `prompt`: Message displayed during pause
- `run_once: true`: Executes only once even when running on multiple hosts

**Why This is Important:**
- OSPF and EIGRP need time to exchange routing information
- Adjacencies need to form before verification
- Without pause, verification commands might run before convergence completes

## Loops and Conditionals

### loop

**Purpose:** Iterates over a list of items, executing the task for each item.

**Usage in Lab:**

**OSPF Network Configuration:**
```yaml
- name: Configure OSPF Network Statements
  cisco.ios.ios_config:
    lines:
      - network {{ item.network }} {{ item.wildcard }} area {{ item.area }}
    parents: router ospf {{ ospf_process_id }}
    save_when: modified
  loop: "{{ ospf_networks[inventory_hostname] }}"
  when: ospf_networks[inventory_hostname] is defined
```

This loops through all networks defined for each router. For example, CSR1 has 4 networks, so this task executes 4 times.

**EIGRP Network Configuration:**
```yaml
- name: Configure EIGRP Process
  cisco.ios.ios_config:
    lines:
      - network {{ item.network }} {{ item.wildcard }}
    parents: router eigrp {{ eigrp_as }}
    save_when: modified
  loop: "{{ eigrp_networks[inventory_hostname] }}"
```

**Accessing Loop Items:**
- `item`: Current item in iteration
- `item.network`: Access dictionary keys (e.g., "11.11.11.0")
- `item.wildcard`: Access dictionary keys (e.g., "0.0.0.255")
- `item.area`: Access dictionary keys (e.g., 51)
- `item.description`: Access dictionary keys (e.g., "To R1")

**Displaying Loop Results:**
```yaml
- name: Display Network Statements Configured
  debug:
    msg: "Network {{ item.item.network }}/{{ item.item.wildcard }} added to Area {{ item.item.area }} - {{ item.item.description }}"
  loop: "{{ network_config.results }}"
  when: 
    - network_config.results is defined
    - item.changed
```

Note: `item.item` is used because we're looping over registered results, which contain the original `item`.

### when

**Purpose:** Conditionally executes a task based on specified conditions.

**Usage in Lab:**

**Check if Variable is Defined:**
```yaml
loop: "{{ ospf_networks[inventory_hostname] }}"
when: ospf_networks[inventory_hostname] is defined
```
This prevents errors if a device has no networks defined.

**Check if Configuration Changed:**
```yaml
- name: Display Router ID Configuration Result
  debug:
    msg: "Router ID {{ router_id }} configured on {{ inventory_hostname }}"
  when: ospf_config.changed
```
Only displays message if configuration actually changed.

**Multiple Conditions (AND):**
```yaml
when: 
  - network_config.results is defined
  - item.changed
```
Both conditions must be true for task to execute.

## Connection Plugin

### network_cli

**Purpose:** Connection plugin specifically designed for network devices that use CLI-based management.

**Configuration in inventory.yml:**
```yaml
all:
  vars:
    ansible_connection: network_cli
    ansible_network_os: cisco.ios.ios
    ansible_user: admin
    ansible_password: Admin@123
    ansible_become: no
```

**What it Does:**
- Establishes SSH connections to network devices
- Handles authentication using username/password
- Manages CLI sessions
- Parses command outputs
- Maintains persistent connections for better performance

**Related Settings in ansible.cfg:**
```ini
[persistent_connection]
connect_timeout = 30
command_timeout = 30
```

These settings control:
- `connect_timeout`: How long to wait for SSH connection (30 seconds)
- `command_timeout`: How long to wait for command response (30 seconds)

## Variables Used in Lab

### Inventory Variables (inventory.yml)

**Global Variables for All Devices:**
```yaml
all:
  vars:
    ansible_network_os: cisco.ios.ios    # Tells Ansible this is Cisco IOS
    ansible_connection: network_cli       # Use CLI connection method
    ansible_user: admin                   # SSH username
    ansible_password: Admin@123           # SSH password
    ansible_become: no                    # Don't use enable mode separately
```

**Host-Specific Variables:**
```yaml
CSR1:
  ansible_host: 11.11.11.11              # Management IP address
  router_id: "255.255.0.1"               # OSPF Router ID
  description: "Area Border Router"      # Device description
```

### Playbook Variables

**OSPF Playbook Variables:**
```yaml
vars:
  ospf_process_id: 1                     # OSPF Process number
  
  ospf_networks:                         # Networks per device
    CSR1:
      - { network: "1.2.3.0", wildcard: "0.0.0.255", area: 0, description: "To SW2" }
      - { network: "192.168.1.0", wildcard: "0.0.0.255", area: 0, description: "To SW3" }
```

**EIGRP Playbook Variables:**
```yaml
vars:
  eigrp_as: 100                          # EIGRP Autonomous System number
  
  eigrp_networks:                        # Networks per device
    SW1:
      - { network: "172.18.1.0", wildcard: "0.0.0.255", description: "To SW2" }
```

**Using Variables in Tasks:**
```yaml
# Access inventory variable
router-id {{ router_id }}

# Access playbook variable
parents: router ospf {{ ospf_process_id }}

# Access loop item
network {{ item.network }} {{ item.wildcard }} area {{ item.area }}

# Access built-in variable
"Configuring: {{ inventory_hostname }}"
```

## Module Documentation Commands

### Viewing Module Help

**List all cisco.ios modules:**
```bash
ansible-doc -l | grep cisco.ios
```

**View detailed documentation:**
```bash
# Full documentation for ios_config
ansible-doc cisco.ios.ios_config

# Full documentation for ios_command
ansible-doc cisco.ios.ios_command
```

**View just the examples section:**
```bash
ansible-doc cisco.ios.ios_config | grep -A 30 "EXAMPLES"
```

**Quick syntax overview:**
```bash
ansible-doc -s cisco.ios.ios_config
```

## Summary

The modules and keywords used in this OSPF/EIGRP lab are:

**Cisco IOS Modules:**
1. **cisco.ios.ios_config** - Configure OSPF and EIGRP
2. **cisco.ios.ios_command** - Run verification commands

**Ansible Core Keywords:**
3. **register** - Store task outputs
4. **debug** - Display information
5. **pause** - Wait for convergence
6. **loop** - Iterate over network lists
7. **when** - Conditional execution

**Connection:**
8. **network_cli** - SSH connection to network devices

These are the ONLY modules and keywords you need to understand to work with the playbooks in this lab.

## Additional Resources

- [Cisco IOS Collection Documentation](https://docs.ansible.com/ansible/latest/collections/cisco/ios/)
- [Ansible Network Automation Guide](https://docs.ansible.com/ansible/latest/network/index.html)
- [Local Module Documentation](Run: `ansible-doc cisco.ios.ios_config`)

## Next Steps

In the next section, we will execute the OSPF and EIGRP configuration playbooks using these modules to automate the multi-area OSPF deployment in the lab environment.