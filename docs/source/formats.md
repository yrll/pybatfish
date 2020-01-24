# Format of vendor and supplemental data

## Vendor formats

For vendors whose device configuration is not a single text file, additional preprocessing may be necessary before uploading data to Batifsh.

### F5 BIG-IP

F5 BIG-IP configuration spans multiple individual files.
Follow the instructions [here](https://github.com/batfish/batfish/wiki/Packaging-F5-Big-IP-configuration-for-analysis) to properly package them for analysis.


### Cumulus Linux

Cumulus devices can be configured by editing individual files, such as `/etc/interfaces`, `/etc/frr.conf` or invoking the [Network Command-Line Utility (NCLU)](https://docs.cumulusnetworks.com/display/DOCS/Network+Command+Line+Utility+-+NCLU)

In either case, follow the instructions [here](https://github.com/batfish/batfish/wiki/Packaging-snapshots-for-analysis#cumulus)

Note: If you are using the `BGP Unnumbered` feature on Cumulus devices, you will need to supply a [Layer1 topology file](#layer-1-topology-file).


## Batfish data formats

### Host JSON files

The host JSON files contain basic information about the hosts attached to the network, including their names, a pointer to their iptables configuration file, and their interfaces. An example file is:

```json
{
  "hostname" : "host1",
  "iptablesFile" : "iptables/host1.iptables",
  "hostInterfaces" : {
    "eth0" : {
      "name": "eth0",
      "prefix" : "2.128.0.101/24"
    }
  }
}
```

`iptables/host1.iptables` is the path relative to the snapshot where this host's iptables configuration can be found. iptables configuration files should be in the format that is generated by `iptables-save`.


### Layer-1 topology file

Normally Batfish infers Layer-3 interface adjacencies based on IP address configuration on interfaces.

For instance, if there are two interfaces in the network with IP assignments `192.168.1.1/24` and `192.128.1.2/24` respectively,
Batfish will infer that these interfaces are adjacent.

However, you may override this behavior by supplying a Layer-1 topology file.
In this case, Layer-3 adjacencies are computed by combining the supplied Layer-1 adjacencies with Layer-2 and Layer-3 configuration to get a more accurate model.
This is especially useful if IP addresses are reused across the network on interfaces that are not actually adjacent in practice.

The expected Layer-1 topology file is a JSON file that has a list of edge records, where each edge record has node and interface names of the two ends.
See [this file](https://github.com/batfish/batfish/tree/master/networks/example/example_layer1_topology.json) for an example.
Your file name should be `layer1_topology.json` for it to be considered by Batfish.


### ISP configuration

Batfish can model routers representing ISPs (and Internet) for a given network.
The modeling is based on a json configuration file (`isp_config.json`),
which tells Batfish about the interfaces on border routers which peer with the ISPs.
An example file is:

```json
{
  "borderInterfaces": [
    {
      "borderInterface": {
        "hostname": "as2border1",
        "interface": "GigabitEthernet3/0"
      }
    }, {
      "borderInterface": {
        "hostname": "as2border2",
        "interface": "GigabitEthernet3/0"
      }
    }
  ],
  "filter": {
    "onlyRemoteAsns": [],
    "onlyRemoteIps": []
  }
}
```

Here `borderInterfaces` contains the list of interfaces on border routers which are meant to peer with the ISPs.
`onlyRemoteAsns` (list of ASNs) and `onlyRemoteIps` (list of IPs) provide a way to apply additional filter by restricting to ISPs having specific ASNs or IPs.

_Batfish will not try to model any ISP routers in the absence of this configuration file._

An example network with ISP modeling configuration is [here](https://github.com/batfish/batfish/tree/master/networks/example/live-with-isp).