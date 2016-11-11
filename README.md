# WP5 

# PSA development and testing instructions
These instructions show how to test PSAs in a local setup and in the NED.

Note: The instructions will be updated later to show how PSAs can be smoke-tested in a real NED environment (possibly run in Jenkins or some staging environment so the testing can be made just by triggering a build on Jenkins after updating PSA codes to the PSA repository). 


## Testing a PSA on a local setup (instructions based on Y1 demo setup)
These instructions can be used to test a PSA in a local setup without the NED inside KVM. These instructions will be updated later. The same setup can be achieved with Virtualbox or VMware.

See the instructions from: https://gitlab.secured-fp7.eu/secured/secured/blob/master/WP5/PSA_local_testing_instructions.docx


### Links to the mentioned images in the guide:
https://www.secured-fp7.eu/files/PSA.img

https://www.secured-fp7.eu/files/TrafficGen.img

https://www.secured-fp7.eu/files/trafficRcv.img 

https://www.secured-fp7.eu/files/PSASchema.json

### Setting up the PSA for testing
Define a suitable way to forward traffic between the interfaces inside your VM. For example, forward the traffic between the interfaces using a bridge (https://wiki.debian.org/BridgeNetworkConnections).

### Running tests
After setting up, you can, e.g., ping the TrafficGen or TrafficRcv to test that the setup is working. After this, you can utilize the already installed tools: Apache server and curl.


## Testing a PSA with the basic NED implementation (v.0.1)
To test a PSA in the actual NED you need to follow the following instructions.

TBD after successfully testing a PSA with the NED.

### Creating a PSA 
#### Using a template
You can use a ready made template image that contain the basic setup needed for a PSA and only modify the PSA mgmt&control to support your PSA, if needed. 

#### Creating a PSA from a clean VM
Package dependencies (required by the PSA mgmt&control)
-TBD

Network interface configuration
-TBD

Updating the latest PSA software to the image
-TBD
