# ansible-letsencrypt-nfsn

A playbook to get an SSL certificate from Let's Encrypt and install it
on a NearlyFreeSpeech.net site using Ansible's letsencrypt module


# Usage


## Copy ansible.cfg to the working directory

I included a copy of `ansible.cfg`. This is the file default file
distributed with Ansible with a few modifications:

* The inventory file is `hosts`, in the current working directory,
* `retry_files_enabled` is `False`,
* The `control_path` is set to `%(directory)s/%%C` because the default
  value creates a string too long for a unix socket name due to the long
  nfsn hostname and username. `%%C` is a hash of all that.


## Install Ansible 2.2

The [letsencrypt module](http://docs.ansible.com/ansible/letsencrypt_module.html)
is available since Ansible 2.2.

I created a Python virtualenv and [installed the latest release version with pip](http://docs.ansible.com/ansible/intro_installation.html#latest-releases-via-pip).


## Fill in the `vars` part of the playbook.

`work_dir` is where you want to create the files on the localhost. Needs
to be an absolute path otherwise the letsencrypt module will be trying
to access a protected directory. I didn't check the code but I suppose
it defaults to a path relative to `/etc`.

`domain` cannot be an array. It would be a nice to have feature, but
let's start with one domain as a string.

`acme_directory` is the staging environment, just like the default. The
variable is a placeholder for you to update it with the production ACME
url when you are ready.

`agreement_url` has been updated since the release of the letsencrypt
module so I put the current version for you here.


## Upload your SSH key to your NearlyFreeSpeech.net profile

Log into NearlyFreeSpeech.net, click on the "profile" tab, click "Add
SSH Key" in the Actions box on the right.


## Create an inventory (`hosts`) file

You need to specify the variables `ansible_host`, `ansible_user` and
`ansible_python_interpreter` for the nfsn environment. For example:

    # hosts
    nfsn ansible_host=<your_nfsn_ssh_hostname ansible_user=<site_memberlogin> ansible_python_interpreter=/usr/local/bin/python


## Run the playbook

Enter your python virtualenv, and run:

    ansible-playbook le-nfsn.yml


## Test, then run against the real ACME Boulder

Boulder is how they call the ACME server. Comment out the
`acme_directory` variable and uncomment the one with the real ACME
directory. You may also want to delete/rename your `cert_file` because
it is still "valid" and the letsencrypt module will not run.


## Cron job

Yet to figure out exactly, but I assume it would require the following:

* A passphraseless SSH key
* A way to enter the virtualenv before calling `ansible-playbook`
* A way to filter out the good output but still send a detailed email
  notification on failure

And I don't think there would be a problem running the cron job
daily or weekly because the letsencrypt module won't do the ACME
challenge (return status "OK") if there is more than 10 days left to the
certificate's expiry date. A copy of the work files must stay within
reach in the `work_dir` though.


## Note on reliability

Sometimes the challenge validation fails, but then I run the playbook
again and it succeeds. I don't know exactly why. Perhaps some HTTP
requests take too long and the server times out.

If that happens you also have to manually clean up and delete the
challenge file. Or not. There is not security issue.


# Copying

By [Alexandre de Verteuil](https://alexandre.deverteuil.net/).

Source hosted on [GitHub](https://github.com/adeverteuil/ansible-letsencrypt-nfsn).

Freely usable, copyable, modifyable and shareable. Please help
yourself. Let me know if you have any improvement ideas!


# References

* http://docs.ansible.com/ansible/letsencrypt_module.html
* https://github.com/abl/letsencrypt-nfsn
* https://members.nearlyfreespeech.net/faq?q=TLSSetup#TLSSetup
