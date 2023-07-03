# Ansible Issue 80233
This project demonstrates ansible [issue
80233](https://github.com/ansible/ansible/issues/80233) when running under molecule.

Running `molecule -vvv converge` fails on checking out ansible v0.0.1:
```
TASK [ansible_issue80233 : Checkout ansible 0.0.1] *****************************
task path: /home/charkins/tmp/ansible_issue80233/tasks/main.yml:26
redirecting (type: connection) ansible.builtin.podman to containers.podman.podman
<instance> RUN [b'/usr/bin/podman', b'mount', b'instance']
<instance> RUN [b'/usr/bin/podman', b'exec', b'instance', b'/bin/sh', b'-c', b'echo ~ && sleep 0']
<instance> RUN [b'/usr/bin/podman', b'exec', b'instance', b'/bin/sh', b'-c', b'( umask 77 && mkdir -p "` echo /root/.ansible/tmp `"&& mkdir "` echo /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733 `" && echo ansible-tmp-1688386451.4328175-1657776-71691746291733="` echo /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733 `" ) && sleep 0']
Using module file /home/charkins/tmp/ansible_issue80233/venv/lib64/python3.11/site-packages/ansible/modules/git.py
<instance> PUT /home/charkins/.ansible/tmp/ansible-local-1656365_1qq_rai/tmptntuwrlx TO /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733/AnsiballZ_git.py
<instance> RUN [b'/usr/bin/podman', b'cp', b'/home/charkins/.ansible/tmp/ansible-local-1656365_1qq_rai/tmptntuwrlx', b'instance:/root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733/AnsiballZ_git.py']
<instance> RUN [b'/usr/bin/podman', b'exec', b'instance', b'/bin/sh', b'-c', b'chmod u+x /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733/ /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733/AnsiballZ_git.py && sleep 0']
<instance> RUN [b'/usr/bin/podman', b'exec', b'instance', b'/bin/sh', b'-c', b'/usr/bin/python3.9 /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733/AnsiballZ_git.py && sleep 0']
<instance> RUN [b'/usr/bin/podman', b'exec', b'instance', b'/bin/sh', b'-c', b'rm -f -r /root/.ansible/tmp/ansible-tmp-1688386451.4328175-1657776-71691746291733/ > /dev/null 2>&1 && sleep 0']
fatal: [instance]: FAILED! => {
    "changed": false,
    "cmd": "/usr/bin/git ls-remote origin -h refs/heads/0.0.1",
    "invocation": {
        "module_args": {
            "accept_hostkey": false,
            "accept_newhostkey": false,
            "archive": null,
            "archive_prefix": null,
            "bare": false,
            "clone": true,
            "depth": null,
            "dest": "/var/lib/issue80233",
            "executable": null,
            "force": false,
            "gpg_whitelist": [],
            "key_file": null,
            "recursive": true,
            "reference": null,
            "refspec": null,
            "remote": "origin",
            "repo": "https://github.com/ansible/ansible.git",
            "separate_git_dir": null,
            "single_branch": false,
            "ssh_opts": null,
            "track_submodules": false,
            "umask": null,
            "update": true,
            "verify_commit": false,
            "version": "0.0.1"
        }
    },
    "msg": "fatal: 'origin' does not appear to be a git repository\nfatal: Could not read from remote repository.\n\nPlease make sure you have the correct access rights\nand the repository exists.",
    "rc": 128,
    "stderr": "fatal: 'origin' does not appear to be a git repository\nfatal: Could not read from remote repository.\n\nPlease make sure you have the correct access rights\nand the repository exists.\n",
    "stderr_lines": [
        "fatal: 'origin' does not appear to be a git repository",
        "fatal: Could not read from remote repository.",
        "",
        "Please make sure you have the correct access rights",
        "and the repository exists."
    ],
    "stdout": "",
    "stdout_lines": []
}

PLAY RECAP *********************************************************************
instance                   : ok=5    changed=4    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

Logging into the container via `molecule login` shows that ansible checked the
repository out as **root**, not **issue80233**:
```
[root@instance ~]# ls -ltr /var/lib/issue80233/
total 64
-rw-r--r--. 1 root root  5697 Jul  3 03:53 README.rst
-rw-r--r--. 1 root root   728 Jul  3 03:53 MANIFEST.in
-rw-r--r--. 1 root root 35149 Jul  3 03:53 COPYING
drwxr-xr-x. 1 root root    94 Jul  3 03:53 changelogs
drwxr-xr-x. 1 root root   298 Jul  3 03:53 bin
drwxr-xr-x. 1 root root    44 Jul  3 03:53 docs
drwxr-xr-x. 1 root root   226 Jul  3 03:53 examples
drwxr-xr-x. 1 root root    14 Jul  3 03:53 lib
drwxr-xr-x. 1 root root   506 Jul  3 03:53 hacking
drwxr-xr-x. 1 root root   132 Jul  3 03:53 licenses
-rw-r--r--. 1 root root  1116 Jul  3 03:53 setup.py
-rw-r--r--. 1 root root  3432 Jul  3 03:53 setup.cfg
-rw-r--r--. 1 root root   995 Jul  3 03:53 requirements.txt
-rw-r--r--. 1 root root   193 Jul  3 03:53 pyproject.toml
drwxr-xr-x. 1 root root    48 Jul  3 03:53 packaging
drwxr-xr-x. 1 root root    64 Jul  3 03:54 test
```