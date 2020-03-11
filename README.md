The network importer is a tool to import/synchronize an existing network with a Network Source of Truth, it's designed to be idempotent and by default it's only showing the difference between the running network and the remote database. 

The main use cases for the network importer 
 - Import an existing network into a SOT (Netbox) as a first step to automate a brownfield network
 - Check the differences between the running network and the Source of Truth

# How to use 

The network importer can run either in `check` mode or in `apply` mode. 
 - In `check` mode, no modification will be made to the SOT, the differences will be printed on the screen
 - in `apply` mode, the SOT will be updated will all interfaces, IPs, vlans etc

## Start Batfish in a container

The network-importer requires to have access to a working batfish environment, you can easily start one using docker
```
docker run -d -p 9997:9997 -p 9996:9996 batfish/batfish:2020.01.11.363
```

## Check/Create your devices in Netbox

A bare device needs to be already present in Netbox and the network-importer will be able to import all vlans, interfaces, ip addresses, cables, transceivers etc ..  
Currently, the network-importer is not creating the devices in Netbox.   

To be able to connect to the device the following information needs to be defined in NetBox:
- Primary ip address
- Platform (must be a valid napalm driver or have a valid napalm driver defined)
> Connecting to the device is not mandatory but some features depends on it: configuration update, transceivers, mostly cabling.

## Configuration file

```toml
[main]
# import_ips = true 
# import_cabling = "lldp"       # Valid options are ["lldp", "cdp", "config", false]
# import_transceivers = false 
# import_intf_status = true     # If set as False, interface status will be ignore all together
# import_vlans="config"         # Valid options are ["cli", "config", true, false]

# nbr_workers= 25

# Not fully fonctional right now, need to revisit that part
# generate_hostvars = false 
# hostvars_directory= "host_vars"

# 
# inventory_source="netbox" # Valid option ["netbox", "configs"]
# inventory_filter= ""

# Directory where the configuration can be find, organized in Batfish format
# configs_directory= "configs"

#  
# data_directory="data"
# data_update_cache=true
# data_use_cache=false

[batfish]
address= "localhost"   # Alternative Env Variable : BATFISH_ADDRESS
# api_key= "XXXX"      # Alternative Env Variable : BATFISH_API_KEY
# network_name="network-importer"
# snapshot_name="latest"
# port_v1= 9997
# port_v2= 9996
# use_ssl= false

[netbox]
# The information to connect to netbox needs to be provided, either in the config file or as environment variables
address = "http://localhost:8080"                   # Alternative Env Variable : NETBOX_ADDRESS
token = "113954578a441fbe487e359805cd2cb6e9c7d317"  # Alternative Env Variable : NETBOX_TOKEN
verify_ssl = true                                   # Alternative Env Variable : NETBOX_VERIFY_SSL
cacert = "/tmp/netbox.crt"                          # Alternative Env Variable : NETBOX_CACERT

# Define a list of supported platform, 
# if defined all devices without platform or with a different platforms will be removed from the inventory
# supported_platforms = [ "ios", "nxos" ]

# Update device configuration on Netbox add the end of the execution
# status_update = false 
# status_on_pass = 1
# status_on_fail = 4
# status_on_unreachable = 0 

[network]
# To be able to pull live information from the devices, the credential information needs to be provided
# either in the configuration file or as environment variables ( & NETWORK_DEVICE_PWD)
login = "username"      # Alternative Env Variable : NETWORK_DEVICE_LOGIN
password = "password"   # Alternative Env Variable : NETWORK_DEVICE_PWD

[logs]
# Define log level, curently the logs are printed on the screen
# level = "info" # "debug", "info", "warning"

# For each run, a performance log is generated by default to capture how long
# some functions took to execute
# performance_log = true
# performance_log_directory = "performance_logs"
            
# When running in Apply mode, all the changes are logged in a changelog file
# the supported format are text and jsonlines
# change_log = true
# change_log_format= "text"  # "jsonlines", "text"
# change_log_filename= "changelog"
```

# How does it work

The network importer is using different tools to collect information from the network devices: 
- [batfish](https://github.com/batfish/batfish) to parse the configurations and extract a vendor neutral data model. 
- [nornir], [naplam], [netmiko] and [ntc-templates] to extract some information from the device cli if available

# disclaimer / Assumption

Currently the library only supports netbox but the idea for 1.0 is to support multiple backend SOT
Currently the assumption is that vlans are global to a site. need to find a way to provide more flexibility here without making it too complex