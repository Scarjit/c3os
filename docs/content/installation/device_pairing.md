+++
title = "Device Pairing"
date = 2022-02-09T17:56:26+01:00
weight = 1
chapter = false
pre = "<b>- </b>"
+++

For pairing a c3os node, you will use the `c3os` CLI which is downloadable as part of the releases from another machine, it will be used to pair and install a new node or join a node to an existing cluster.

## Start the c3os ISO

Download and mount the ISO in either baremetal or a VM that you wish to use as a node for your cluster.

It doesn't matter if you are joining a node to an existing cluster or creating a new one, the procedure is still the same.

A GRUB menu will be displayed:

![VirtualBox_test22_10_02_2022_20_56_55](https://user-images.githubusercontent.com/2420543/153488323-1ab451c3-d6ef-4109-b535-be8a823ba356.png?classes=border,shadow)

The first menu entry starts `c3os` in **Decentralized Device Pairing** pairing mode and is the default, while the second is reserved for manual installations.

Once booted the first entry, a boot splash screen will appear, and right after a QR code will be printed out of the screen
![VirtualBox_test22_10_02_2022_20_56_29](https://user-images.githubusercontent.com/2420543/153488315-a4290028-b856-436d-a43a-ea0404003fdf.png?classes=border,shadow)

## Prepare a configuration config file

In the machine you are using for bootstrapping (your workstation, a jumpbox, or ..)

Create a config file like the following, for example `config.yaml`:

```yaml
stages:
   network:
     - name: "Setup users"
       authorized_keys:
        c3os: 
        - github:mudler
c3os:
  network_token: "...."
  # Optionally, set a network id (for multiple clusters in the same network)
  # network_id: "dev"
  # Optionally set a role
  # role: "master"

```

{{% notice note %}}
If you are creating a new cluster, you need to create a new network token with the `c3os` CLI: `c3os generate-token`
{{% /notice %}}


The configuration config file is in [cloud-init](https://rancher.github.io/elemental-toolkit/docs/reference/cloud_init/) syntax and you can customize it further to setup the machine behavior.

## Pair the machine

The VM once finished booting will print-out a QR code like the following:

![VirtualBox_test22_10_02_2022_20_56_36](https://user-images.githubusercontent.com/2420543/153488321-07e63e5f-d9e3-48ce-b551-8b457ece14a9.png?classes=border,shadow)


You can use now the QR code to pair the machine by either providing a screenshot or photo of it, or by just calling `c3os register` which will take a screenshot by default:

```
c3os register --reboot --device /dev/sda --config config.yaml
```

We can also specify here if the machine after install needs to be rebooted (`--reboot`) or shut down (`--poweroff`). The cloud-init configuration file must be provided with the `--config` flag.

Optionally we can specify an image where to extract the QR code from, by specifying an image file as argument:

```
c3os register --device /dev/sda --config config.yaml <file.png>
```

At this point, wait until the pairing is complete and the installation will start automatically in the new node.

## Join new nodes

To join new nodes, simply re-apply the process to new nodes by specifying the same `config.yaml` for all the machines. The machines will connect automatically between themselves, either remotely on local network.

## Get kubeconfig and connect to the nodes

In the machine you are using for bootstrapping (your workstation, a jumpbox, or ..) run in a new terminal, and leave it open (CTRL+C to abort):

```bash
c3os bridge --network-token <TOKEN>
```

This command will create a tun device in your machine and will make possible to contact each node in the cluster.

An API is also available at [localhost:8080](http://localhost:8080). 

After a few moments of bootstrapping, you should be able to see your nodes in the "Machine" tab.

In an new terminal window run:

```bash
c3os get-kubeconfig > kubeconfig
```

You should be now able to use the kubeconfig file as usual.

{{% notice note %}}
`c3os bridge` acts like `kubectl proxy`. you need to keep it open to operate the kubernetes cluster and access the API.
{{% /notice %}}