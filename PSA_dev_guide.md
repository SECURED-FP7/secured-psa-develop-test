# PSA Developer guide for PSA-PSC API v0.5 and NED v0.6

Simle PSA API, without any publish/subscribe support.

## Prerequisites
0. You have (debianPSC.img and) veryLightPSA.img downloaded from https://vm-images.secured-fp7.eu/images/priv/.
+ Credentials to vm-images: username: secured, password: ask WP leader.
+ To reach the host, it's necessary to add in your /etc/hosts file following entry
   + 130.192.56.44 vm-images.secured-fp7.eu vm-images
+ In order to upload files you need to send your SSH pubkey to WP5/WP6 leader.

1. Checkout the PSA-PSC API or the example PSA codes for this version (https://gitlab.secured-fp7.eu/secured/ned/tree/master/PSA, https://gitlab.secured-fp7.eu/secured/ned/tree/master/PSC)

### Credentials for sample VM images (just for development purposes, for validation etc., has to be modified)
+ debianPSC.img: username: root, password: secured
+ veryLightPSA.img: username: root 


## Configuration files / PSA Manifest

__PSAManifest/[psa_id]__

1. Modify the PSA_id according to your [psa_id] and add  the "disk" image of your PSA here. 

```
{
	"PSA_id":"12345",
	"disk": "veryLightPSA.img",
	"interface": [
			{
			"network":"data", 
			"type":"data_in"
			},
			{
			"network":"data", 
			"type":"data_out"
			},
			{
			"network":"control", 
			"type":"manage"
			}
		],
	"memory": "256",
	"os-architecture": "x86_64",
	"vcpu": "1"
}
```


__psaConfigs/[folder: psa_id]/[config_files]__
1. Add sample configurations to be used in your PSA. You can name then as any, or incorporate one for a user, for the first version.


__userGraph/[userx]__
2. Create a new user under this folder with the your new PSA id and add some configuration under security_controls conf_id you created earlier in the psaConfigs/[psa_id] folder.
```
"PSASet": [
		
		{
			"id": "12345",
			"security_controls": [
				
				{
					"imgName": "iptables.img",
					"conf_id":"iptables_user1"
				}
				
			] 
			
		},
        {
	]
```

## PSA API implementation

### __REST calls PSA --> PSC__
1. Once PSA boots, it calls PSC to notify it is online AND retrieves initial configuration

```
POST: PSC_ADDRESS + /v0.5/getConf/{PSA_ID}
Any other PUT: response HTTP 404
```

### __REST calls PSC --> PSA__
2. Once PSA is online, the PSC makes the following REST calls to the PSA to control it.


```
POST: PSA_ADDRESS + /execInterface/v0.5/init?dpid=10.0.0.2&dpgateway=10.0.0.253&dpdns=8.8.8.8
 - Response type: application/json
 - Response: {"command":"init", ,"ret_code":"1"}
 - Response code OK: HTTP 200
```
The NED will dynamically assign an IP for the parameter, the one above is just an example.
The PSA takes the dpid paramter and applies it on the appropriate interface if it needs an IP.
The PSA can choose to ignore **dpid**, **dpgateway** and **dpdns** if it they are not required.


```
POST: PSA_ADDRESS + /execInterface/v0.5/start
 - Response type: application/json
 - Reponse: {"command":"start", "ret_code": {echo of start.sh}}
 - Response code OK: HTTP 200
```

```
POST: PSA_ADDRESS + /execInterface/v0.5/stop
 - Response type: application/json
 - Reponse: {"command":"stop", "ret_code": {echo of stop.sh}}
 - Response code OK: HTTP 200
```

```
GET: PSA_ADDRESS + /execInterface/v0.5/status
 - Response type: application/json
 - Reponse: {"command":"status", "ret_code": {echo of status.sh}}
 - Response code OK: HTTP 200
```

```
GET: PSA_ADDRESS + /execInterface/v0.5/configuration
 - Response type: application/json
 - Reponse: {"command":"configuration", "ret_code": {echo of current_config.sh}}
 - Response code OK: HTTP 200
```

```
GET: PSA_ADDRESS + /execInterface/v0.5/log
 - Reponse: PSA log file from the PSA_log_location
 - Response type: text/plain
 - Response code OK: HTTP 200
 - Response code no log available: 501
```
```
Error: Response code: HTTP 501
Any other GET: 404
```



The simple PSA API only requires the developer to do the following:
1. __Modify the necessary scripts under PSA/scripts.__
2. Modify PSA/boot_script_psa, if necessary.
3. Modify PSA/interfaces, if necessary
4. Modify psaEE.conf
5. Upload these files to the VM image with all the required PSA libraries pre-installed.
6. Upload the ready PSA VM image to https://vm-images.secured-fp7.eu/images/priv/

###  The following shows an example of the script files with the sample PSA implementation (iptables).

1. init.sh

    + https://gitlab.secured-fp7.eu/secured/ned/blob/master/PSA/scripts/init.sh
    + Configures / initializes the PSA. This script is called after the configuration / PSA configuration blob.

2. current_config.sh

    + https://gitlab.secured-fp7.eu/secured/ned/blob/master/PSA/scripts/current_config.sh
    + Returns the currently active configuration of the PSA, in free text format.

3. start.sh

    + https://gitlab.secured-fp7.eu/secured/ned/blob/master/PSA/scripts/start.sh
    + Starts the PSA. 

4. status.sh

    + https://gitlab.secured-fp7.eu/secured/ned/blob/master/PSA/scripts/status.sh
    + Returns the status of the PSA. 1 if OK/running. 0 otherwise.

5. stop.sh

    + https://gitlab.secured-fp7.eu/secured/ned/blob/master/PSA/scripts/stop.sh
    + Stops the PSA. Currently still allows data traffic.


You need to modify the psaEE.conf with your PSA configuration. You need to modify at least: psa_id, psa_name, psa_version, psa_api_verison and psa_log_location.

```
[configuration]
psc_address=http://192.168.2.1:8080
psa_config_path=/home/psa/pythonScript/psaConfigs
scripts_path=/scripts/
psa_id=12345
psa_name=Simple iptables PSA
psa_version=0.1.0
psa_api_version=v0.5
psa_log_location=/home/psa/pythonScript/psaConfigs/psa.log
conf_id=
```


## Running/testing with the NED v0.6

1. In order to test this, you have to copy these folders/files onto your NED folder.

+ PSAManifest/[psa_id]
+ psaConfigs/[psa_id]/[config_files_for_user]
+ userGraph/[user_X]

2. Copy your PSA VM image into /var/lib/libvirt/images/ folder.

3. Run setup.sh from the NED and log in as the user_x you just created.
