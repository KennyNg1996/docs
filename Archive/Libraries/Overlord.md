# Overlord Deployment and Monitoring Library

Overlord is a library for deploying an arbitrary Repy program on a number of vessels. Built on top of the [ExperimentLibrary Experiment Library], it persistently manages a user-defined number of vessels, ensuring that the specified service is up and running.



## Setup

To begin setup, create a new directory of Overlord:
```
mkdir overlord
```

Overlord requires the Seattle Experiment Library, which can be obtained by following the steps on [wiki:Libraries/ExperimentLibrary the associated wiki page]. Note that the ```experimentlibrary``` directory must be a direct subdirectory of the ```overlord``` directory created above.

Also note that Overlord depends on Experiment Library functions that make secure connections to Seattle Clearinghouse. To perform secure SSL communication with Seattle Clearinghouse, you must:
  * Have a GENI user's public and private key files in the same directory as the application incorporating Overlord's API
    * The filenames must be of the form "USERNAME.publickey" and "USERNAME.privatekey"
  * Have [M2Crypto](http://chandlerproject.org/Projects/MeTooCrypto) installed
  * Have a PEM file, (with filename cacert.pem) containing CA certificates in the experimentlibrary directory. One such file can be found at http://curl.haxx.se/ca/cacert.pem

Once all of these dependencies are squared away, you can get a copy of Overlord by running the following command:
```
svn export https://seattle.poly.edu/svn/seattle/trunk/overlord/overlord.py
```




## Usage

Using the API provided by Overlord, you can create simple Python scripts to deploy services on vessels.

To create an Overlord client, simply import it:                               
```
import overlord                                                             
```

and then create an Overlord object with your desired parameters:

```
example = overlord.Overlord(GENI_USERNAME, VESSEL_COUNT, PROGRAM_FILENAME, LOG_FILENAME)
```

`init()` performs all of the initial configuration of vessel deployment. It takes four arguments:
  i. A valid Seattle Clearinghouse username
  ii. The number of vessels on which to deploy
  iii. The type of vessels on which to deploy (wan, lan, etc)
  iv. The filename of the Repy program to deploy
  v. (optional) The file that Overlord will log to
    * Passing no argument or 'None' will simply have Overlord log directly to console

`run()` initiates deployment and runs forever, maintaining the user-defined number of deployed vessels. Any arguments passed to run() will be passed directly to the Repy program run on vessels. For example, many services require an argument local port on which to listen for connections. For such services, the Seattle Clearinghouse user's local port (available within the config dictionary of the Overlord object) must be passed to `run()`.

Once `run()` is called, Overlord will deploy the Repy program on the specified number of vessels and periodically check to ensure that the service is up and running.

The service can be stopped by simply creating and naming a file ```stop``` within the folder of the program using Overlord's API, taking at most 15 minutes by default to terminate.


### Overriding run()'s Default Behavior
With some exceptions, most of run()'s functionalities are made up of a series of separate method names represented by class variables. While the default methods are sufficient for basic functionality in keeping a repy service up and running, they can be overridden to suit an application's individual needs. They consist of:
 * init_overlord_func - run()'s start-up operations
 * acquire_vessels_func - how run() acquires new vessels
 * init_vessels_func - how run() initializes the vessels
 * remove_vessels_func - how run() weeds out certain vessels 
 * maintenance_func - how run() maintains user's ownership of existing vessels

To override the default methods from outside of overlord.py, simply set Overlord's corresponding class variable to the name of the replacement method. Override methods, however, must abide to certain restrictions in order to function correctly with run()'s behaviors that have been left untouched.

Details on each functions default behavior and restrictions on override methods can be found in Overlord's [wiki:Libraries/Overlord#Documentation Documentation]
 



## Example
The following script will deploy NTP time servers on 10 WAN vessels:
```
import overlord
time_server = overlord.Overlord(GENI_USERNAME, 10, 'wan', 'time_server.py')
time_server.run(timeserver.config['geni_port'])
```

Note that `time_server.py` requires a local port number as an argument, so the user's GENI port is passed to the `run()` function.

The following script will replace run()'s start-up behavior in releasing currently owned vessels with a method that prints 'foo':
```
import overlord
def print_foo(overlord):
  print 'foo'

time_server = overlord.Overlord(GENI_USERNAME, 10, 'wan', 'time_server.py')
time_server.init_overlord_func = print_foo
time_server.run(timeserver.config['geni_port'])
```
It is very important to set the override method to the Overlord object's class variable before executing run()



## Documentation

Please refer to [browser:seattle/trunk/overlord/overlord.py the docstrings and comments in overlord.py] for further documentation.