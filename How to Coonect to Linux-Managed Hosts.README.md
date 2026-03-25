Allow the control node (your Ansible machine) to SSH into all 10 managed hosts without a password, using SSH keys for the user ansibleu.

Step‑by‑Step Setup
1️ Create the user on all managed hosts
If the user doesn’t exist yet:

sudo useradd -m -s /bin/bash ansibleu
sudo passwd ansibleu   # optional if you want a fallback password
If you want this user to run sudo without password:

echo "ansibleu ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/ansibleu
2️ Generate SSH key on the Ansible control node
Run this on your control machine:

ssh-keygen -t ed25519 -C "ansible key"
Press Enter for default path (~/.ssh/id_ed25519) and no password.

3️ Copy the public key to each managed host
You can do it manually:

ssh-copy-id ansibleu@host1
ssh-copy-id ansibleu@host2
...
ssh-copy-id ansibleu@host10
Or automate it with a loop:

for host in host1 host2 host3 host4 host5 host6 host7 host8 host9 host10; do
    ssh-copy-id ansibleu@$host
done
This creates:

/home/ansibleu/.ssh/authorized_keys
with correct permissions.

4️ Verify password‑less login
Test:

ssh ansibleu@host1
If it logs in without asking for a password, you're good.

Do it the Ansible way (recommended)
Create a simple playbook:

---
- hosts: all
  become: yes
  tasks:
    - name: Ensure ansibleu exists
      user:
        name: ansibleu
        shell: /bin/bash
        create_home: yes

    - name: Install authorized key
      authorized_key:
        user: ansibleu
        state: present
        key: "{{ lookup('file', '~/.ssh/id_ed25519.pub') }}"
Run it:

ansible-playbook setup-ansible-user.yml
This is cleaner, repeatable, and exactly the kind of automation you prefer.

