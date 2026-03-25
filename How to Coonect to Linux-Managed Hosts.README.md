

#  Goal  
Enable **password‑less SSH** from the control node → all managed hosts  
Using user: **ansibleu**

---

#  Step 1 — Ensure the user exists on all managed hosts  
If the user is not created yet, run this manually **or** via Ansible.

### Manual (per host)
```bash
sudo useradd -m -s /bin/bash ansibleu
sudo passwd ansibleu   # optional
```

### Allow password‑less sudo (recommended for automation)
```bash
echo "ansibleu ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/ansibleu
```

---

# Step 2 — Generate SSH key on the control node  
Run this **only once** on your Ansible machine:

```bash
ssh-keygen -t ed25519 -C "ansible key"
```

Press Enter for all defaults.

This creates:

- `~/.ssh/id_ed25519`
- `~/.ssh/id_ed25519.pub`

---

#  Step 3 — Copy the public key to all managed hosts  
You can do it manually:

```bash
ssh-copy-id ansibleu@192.168.2.194
ssh-copy-id ansibleu@192.168.3.194
...
ssh-copy-id ansibleu@192.168.10.194
```

### Or automate it with a loop (much cleaner)

```bash
for i in {2..10}; do
    ssh-copy-id ansibleu@192.168.$i.194
done
```

This creates:

```
/home/ansibleu/.ssh/authorized_keys
```

with correct permissions.

---

#  Step 4 — Verify password‑less login  
Test one host:

```bash
ssh ansibleu@192.168.2.194
```

If it logs in without asking for a password → success.

---

#  The Ansible Way (recommended for your lab style)

This is the cleanest, most reproducible method — and it matches your preference for automation and maintainability.

### **Playbook: setup-ansible-user.yml**

```yaml
---
- hosts: all
  become: yes

  tasks:
    - name: Ensure ansibleu exists
      user:
        name: ansibleu
        shell: /bin/bash
        create_home: yes

    - name: Allow passwordless sudo
      copy:
        dest: /etc/sudoers.d/ansibleu
        content: "ansibleu ALL=(ALL) NOPASSWD: ALL\n"
        mode: '0440'

    - name: Install authorized key for ansibleu
      authorized_key:
        user: ansibleu
        state: present
        key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"
```

### Run it:

```bash
ansible-playbook -i hosts setup-ansible-user.yml
```

This will:

- Create the user  
- Set up sudo  
- Push your SSH key  
- Fix permissions  
- Make the environment consistent across all hosts  

Exactly the kind of reproducible workflow you prefer.

---

