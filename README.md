# Vagrant Chef Lab

`Vagrant Chef Lab` is a way to stand up an Automate instance and 2 Linux virtual machines using Vagrant and
VirtualBox.

## Chef Compliance Demo
Additionally, in this version Chef Habitat is installed on the Linux VMs and the `chef-cis-sample-linux-inspec`
package is loaded to demonstrate the [Chef Compliance] (https://docs.chef.io/compliance/) audit sample.  Remediation is not included in this example.

## Environment Set-up and Execution

1. Install [Chef Workstation](https://downloads.chef.io/chef-workstation)
2. Install [git](https://git-scm.com/downloads)
3. Install [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
4. Install [Vagrant](https://www.vagrantup.com/downloads.html)
5. Clone this repo
6. Add `192.168.33.199 chef-automate-test.net` to your workstation hosts file

Once you have all the above steps completed, from the root of the cloned repo:

Execute `vagrant up`

Once the virtual machines are up, the credentials to login to Automate are contained in
a file output from Vagrant in the repo's root directory named `automate-credentials.toml`

Using those credentials, login to [https://chef-automate-test.net](https://chef-automate-test.net)
(Note: you'll need accept the risk for the self-signed cert)

Enter the form data to activate the 60 trial license and then navigate to the Compliance tab where
you'll see something like this:

![Chef Automate image 1](images/automate-01.png)

![Chef Automate image 2](images/automate-01.png)

## Using Waivers

Available in Chef InSpec verion 4.18.104 and later, Waivers fulfill a purpose of improving skipped controls by allowing the ability to provide a business justification for controls against which they are unable to be compliant. They can also specify an end date to track when a control should be remediated, or leave it blank to make the waiver permanent.  There are different ways to utilize Waivers based on the context of InSpec execution. In the context of this example using Chef Habitat, a `waiver.toml` is included in this repo.  Copy the `waiver.toml` to the VM, log in to the VM and apply the waivers to the audit service running under Chef Habitat using the following command issued in the path of the `waivers.toml`:

`sudo hab config apply chef-cis-sample-linux-inspec.default $(date +%s) waivers.toml`

The next time the Chef InSpec scan runs, the InSpec scan results will show a compliant result...well, unless you try this after the expiration date in the `waivers.toml`.
