![hpedevlogo2](https://github.com/Workshops-on-Demand/.github/assets/25387895/05d10b28-4447-4b94-8169-f79e617ccde7)

# Workshops-on-Demand
The Workshops-on-Demand infrastructure is currently based on virtual machines. By default, it requires 3 different virtual machines.

Here are a series of blogs published on the [HPE Developer Community Portal](https://developer.hpe.com/blog) describing the genesis of the project. It also covers the deloyment phase of the different components as well as the creation of content.

* 1/ [Open sourcing Workshops-on-Demand part 1: Why and How](https://developer.hpe.com/blog/willing-to-build-up-your-own-workshops-on-demand-infrastructure/)
* 2/ [Open Sourcing Workshops-on-Demand part 2: How to Deploy the backend](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part2-deploying-the-backend/)
* 3/ [Open Sourcing Workshops-on-Demand part 3: Understand the Backend](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part3-understanding-the-backend/)
* 4/ [Open Sourcing Workshops-on-Demand part 4: Manage the Backend](https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part4-managing-the-backend/)
* 5/ Open Sourcing Workshops-on-Demand part 5: How to Deploy the frontend and API servers (To be Published)
* 6/ Open Sourcing Workshops-on-Demand part 6: Create new workshops (To be Published)

# WoD infrastructure functions and repositories

The WoD infrastructure comprises 3 differents systems to work, that are usually spread across 3 machines:
 
* a wod-backend machine, hosting the Jupyter Hub and the WoD templates to generate the real workshop that a given student will run, witth eir metadata. This machine may also intercat with appliances for WoD neededd one, such as Docker e.g. You may have multiple wod-backend in case of a large setup. Corresponding software on the repos [wod-backend](https://github.com/Workshops-on-Demand/wod-backend), [wod-notebooks templates](https://github.com/Workshops-on-Demand/wod-notebooks), [wod-private optional setup](https://github.com/Workshops-on-Demand/wod-private)
* a wod-api-db machine hosting the WoD API service and a PostgreSQL database to store live information about the running platform. Corresponding software on the repo [wod-api-db](https://github.com/Workshops-on-Demand/wod-api-db)
* a wod-fronted machine, hosting the Web interface to see the list of available workshops and book one. Corresponding software on the repo [wod-frontend](https://github.com/Workshops-on-Demand/wod-frontend)

# Installing a WoD infrastructure

In order to install a full infrastructure, a set of reliminary steps are required:

* Install a VM/Physical machine with [Ubuntu 24.04 LTS](https://www.ubuntu-fr.org/download/) minimal and default setup. (20.04 or 22.04 should still work, while less tested these days).
* On each VM the user ubuntu is being created and can be used for the initial setup. For access to the machine and account refer to the [Ubuntu documentation](https://ubuntu.com/server/docs/basic-installation) or how a VM template was set up. Ensure that this user as root access (either via sudo or with password access)
* Then you have to ensure minimal dependencies are present to run the installation. We only need git. All other dependencies are installed by the installer. So issue on each machine: 
        sudo apt install git
* Once installed you can use it to clone the installer (part of the [backend project](https://github.com/Workshops-on-Demand/wod-backend)) again on each machine:
        git clone https://github.com/Workshops-on-Demand/wod-backend
* Then you use the installer to install your WoD infrastructure:
        cd wod-backend/install ; ./install.sh -h
* Reading the exmaple for the full infrastructure at the end of the help message should guve you the required guidance to set it up.

Note that the installer should support Rocky/Alma or Debian fairly easily instead of Ubuntu, but tests have not been fully performed. Contributions welcome.
