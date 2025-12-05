![Frontend](img/wod-frontend.png "Frontend")

Leveraging the JupyterHub technology, the HPE DEV team, at the origin of the project, created a set of hands-on technology training workshops where anyone could connect to our service and benefit from notebook-based workshops at any time, and from anywhere. Read our [User's Guide](USER-GUIDE.md) to see it in action.

Now, let me explain how we made this possible…

# Where it all comes from

We built up our first JupyterHub server for the virtual HPE Technology & Solutions Summit (TSS) that took place in March 2020 and used it to deliver the workshops we would normally give in person in a virtual way. We hosted just a few Jupyter notebooks-based workshops and found that they scored really well in feedback we received from the students. All notebooks were delivered through a single instance of a JupyterHub server.

During the event, we ran only a single workshop at a time. We first set up a range of users (students) to match a given workshop. It required a lot of manual setup, a few scripts and a single Ansible playbook to handle the copy of the workshop content assigned to the desired range of students, with a few variable substitutions like student IDs, passwords, and some API endpoints when relevant. The student assignment was done manually, at the beginning of every workshop. For instance, Fred was student1, Didier was student2, and so on… which was quite cumbersome. When you only have a range of 25 students, one or two people handling the back end is sufficient. But if 40 people show up, then it becomes tricky.

This initial work lead us the following results collected in our dashboard presenting:
-    The number of Active Workshops
-    The active workshops by type
-    The total number of registrations from November 1st 2020 till March 30th 2021
-    The total number of Customer (student) registrations
-    The total workshops split
![Hackshack Dashboard](img/hackshack-dashboard.png "Hackshack Dashboard")

As you can see, moving from a heavily manually oriented approach to now a fully automated one helped increase our numbers. By the end of 2022, more than 4000 had registered for our workshops.

Of course, as automation minded people, we could not stay like that for long, so we decided to build a framework to help us deliver more Workshops on Demand (aka WoD) and began our 5 years journey of developing it till version 1.0.0 :-)

# Architecture and technologies considerations

Soon we realized that for the project to be usable, we would need to improve various aspects. Our needs were the follwing:
- provide a registration portal to allow workshop exposure and automated user registration. 
- automate fully all the tasks on the platform (deployment of the notebook, once chosen on that portal, cleanup, security, ...)
- keep track of the association made between users and students, the workshop chosen, 
- manage workshops (capacity, reset specificities, ...)
- add easily more content to the platform, developed by contributors
- manage a private set of data (content, parameters, scripts) in addition to the public one provided
- support mulitple locations for redundancy, technology specificities
- operate multiple platforms (dev, staging, production, ...)
- operate as well on VMs or physical servers
- deploying the platform from zero to ready to operate fully automated

As you can see, we have a few environments to deploy and manage. The various aspects were coded during 5 years to reach the desired state expressed upper. Without automation, it would have been too much work and prone to error. That was a key aspect of our approach.

We quickly chose the various tools that will help us manage this efficiently:
- Linux for the OS ([Ubuntu LTS](https://ubuntu.com/download/server) being the primary distribution, CentOS/[Rocky Linux](https://rockylinux.org/download) the alternative for backend)
- [Ansible](https://en.wikipedia.org/wiki/Ansible_(software)) for automation and conformity management
- [YAML](https://en.wikipedia.org/wiki/YAML) for configuration files (wod.yml, ansible playbooks and variables)
- [Git](https://en.wikipedia.org/wiki/Git) as a configuration management system storing all our files and [GitHub](https://github.com/Workshops-on-Demand) for publication 
- [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) to store permanently some information around users, workshops, students, ...
- [Javascript](https://en.wikipedia.org/wiki/JavaScript), [NodeJS](https://en.wikipedia.org/wiki/Node.js), [React](https://en.wikipedia.org/wiki/React_(software)), [Grommet](https://v2.grommet.io/) for the frontend and API servers
- [REST](https://en.wikipedia.org/wiki/REST) API (The one from JupyterHub, and one we created for our project)
- [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) API using [procmail](https://en.wikipedia.org/wiki/Procmail) for our incoming communication with the backend

From an architecture perspective, we rapidly agreed that we would need multiple machines/services:
![Multiple servers/services](img/wod-infra-open-source.png "Multiple servers/services")
- A frontend server to manage the workshop list and allow user registration
- An API server coupled with the Database to orchestrate the platform
- A backend server on which will run the JupyterHub service and where the Notebooks will be deployed
- Appliances servers running the technology at work in the Notebook (optional)

For our automation, we have Ansible playbooks that would perform:
- Server configuration (based on an existing deployed Operating System with just git installed) including:
    - Repository Update
    - Apps Installation
    - System Performance Tuning including kernels setup & configuration 
    and depending on the server:
    - on backend:
        - JupyterHub application installation and configuration
        - Linux students creation
        - JupyterHub users creation
        - A complete JupyterHub server preinstalled and configured with some additional Jupyterhub kernels
        - A postfix server used for the procmail API 
    - on frontend:
         - Javascript setup with npm
         - portal installation and launch
    - on the API/DB:
         - Javascript setup with npm
         - PostgreSQL install and setup with sequelize
         - REST API server installation and launch
- Server conformity management (run after the previous playbook and on demand and nightly to ensure that the configuration is consistent and up to date):
    - System Update
    - Security Setup
    - Services Conformity and check (apps, users)
    - Templating of configration and data files
    - A fail2ban service  to limit abuses
    - An Admin user to manage everything 
- Notebooks deployment on the JupyterHub server:
    - Templating the Notebook and data files
    - Security setup


The Workshops-on-Demand GitHub repositories can be found [here](https://github.com/Workshops-on-Demand/).

We created as many Git repositories as needed, for the infrastructure management:
![WoD repositories](img/wod-repositories.png "WoD repositories")
- [wod-frontend](https://github.com/Workshops-on-Demand/wod-frontend) for the WoD portal based on NGINX and NodeJS technologies, it provides the participtants' Registration Portal used to enable booking of the workshops.
- [wod-api-db](https://github.com/Workshops-on-Demand/wod-api-db) for the REST Open API 3.0 based API and a PostgreSQL DB hosting the different status of participants, workshops, and students. 
- [wod-backend](https://github.com/Workshops-on-Demand/wod-backend) for the JupyterHub and Postfix mail server.
- [wod-notebooks](https://github.com/Workshops-on-Demand/wod-notebooks) for the public Jupyter based Notebooks provided. You can test them live at <https://hackshack.hpedev.io/workshops>
- [wod-private](https://github.com/Workshops-on-Demand/wod-private) as a template for using a WoD private setup alongside the public one, including a cutomization layer on top of the public standard WoD Backend / WoD Notebooks content. Do not put any confidential data here as this is a public repository!
- [wod-install](https://github.com/Workshops-on-Demand/wod-install) for the installation of the WoD infrastructure, allowing you to install either Backend, API-DB or Frontend servers using a single line of command.
- [wod-doc](https://github.com/Workshops-on-Demand/.github) for the project documentation (you're on it !)

**Note**: There are 7 repositories available for now. 

# How it works

Let us first show you how the overall process works for our Workshops-on-Demand:

![Workshops-on-Demand principles](img/wod-principles.png "Workshops-on-Demand principles")

If you’re looking for a live explanation of this automation, you can review [the following session](https://www.youtube.com/watch?v=D6Ss3T2p008&t=515s) delivered at Linuxconf in Australia in January 2021. If you prefer reading, consider our [User's Guide](USER-GUIDE.md). Here we'll go into more technical details on how this performed:

**Step 1:** The customer registers for a workshop online at our portal. When clicking the register button for the selected workshop and after agreeing to the terms and conditions, the frontend triggers the first REST API calls to the API-DB server:
-    Register the student in the Customers Database.
-    Send a welcome email to the student.
-    Assign a student ID to him/her according to the student range allocated to the given workshop.

**Step 2:** The API-DB server then orders (through a mail API call managed by procmail) the backend to:
-    Generate a random password for the selected student
-    Deploy the selected workshop

**Step 3:** The backend server calls back the API-DB server using its REST API to:
-    Provide back the new student password
-    Make the student’s database record active
-    Decrement Workshop capacity

**Step 4:** The API-DB server sends:
-    The credentials email to allow the student to connect to the workshop

This infrastructure automation relies mainly on a few bash scripts and Ansible playbooks to: 
-    Generate random passwords
-    Update LDAP passwords accordingly when required
-    Deploy the given workshop to the proper student home directory through an Ansible playbook
-    Update the notebook content through Ansible variables substitutions
     -    Student ID
     -    Student password
     -    API Endpoints definition
-    Update student permissions
-    Perform all necessary create or reset actions linked to a given workshop outside of the notebook customization

The API-DB server actually sends an email to the backend with a dedicated format (an API!) that is then parsed by the Procmail process to retrieve the necessary information to perform the different tasks. We use 4 verbs: CREATE (to setup to the student environment as described upper), DELETE (to remove the user from tables), RESET (to clean up the student content and reset the back-end infrastructure when needed) and PURGE (globally DELETE+RESET aka full cleanup)

At the end of this automated process, the backend makes a series of API calls to the API-DB server to send back the required information, like a new password, workshop status, etc.

# A bit of background

This solution allows us to deliver free hands-on workshops. It has been used during events (HPE Discover, HPE TSS, as well as Open Source Summit, KubeCon), and during training sessions (HPE trainings, RMLL, Campus Numérique, ...) to promote API/automation-driven solutions along with some 101-level coding courses.

If you are interested in creating your own training architecture to deliver workshops, this project is definitely for you. It would allow you to create, manage content and deliver it in an automated and very efficient way. Once ready, the infrastructure can deliver workshops in matter of minutes!

Trying to build a similar architecture on your own is obviously possible, but we integrated so much automation around the different components like the JupyterHub server deployment along with multiple pre installed kernels, user management and much more. When leveraging our project, one can actually get a proper public-based environment in 2 hours.

## Why would we consider open sourcing our Workshops-on-Demand?

Firstly, if you read carefully the messaging on our [homepage](https://developer.hpe.com/) , you will find words like sharing and collaborating. This is part of the HPE Developer team's DNA. 

Secondly, the project is based on open source technologies like Jupyter and Ansible. It felt natural that the work we did leveraging these should also benefit the open source community.

We have, actually, shared the fundamentals of the project thoughout the HPE Developer Community, and to a wider extent, the Open Source Community  through different internal and external events. And the feedback has always been positive. Some people found the project very appealing. Originally, and long before even thinking of open sourcing the project, when we really started the project development, people were mainly interested in the content and not necessarily in the infrastructure. The students wanted to be able to reuse some of the notebooks. And in a few cases, they also asked for details about the infrastructure itself, asking about the notebooks delivery mechanism and other subjects like the [procmail API](https://www.youtube.com/watch?v=zZm6ObQATDI).

In 2022, we were contacted by an HPE colleague who was willing to replicate our setup in order to deliver notebooks to its AI/ML engineers. His aim was to provide a simple, central point from which he could deliver Jupyter Notebooks, that would later be published on the Workshops-on-Demand infrastructure frontend portal, allowing content to be reused and shared amongst engineers. While, over time, we had worked  a lot on automating content delivery and some parts of the infrastructure setup, we needed now to rework and package the overall solution to make it completely open source and reusable by others.

As a result of our work on that project, over the course of 2022 we started to open source the Workshops-on-Demand program. As a project developed within the confiines of Hewlett Packard Enterprise (HPE), we had a number of technical, branding, and legal hurdles we needed to overcome in order to achieve this.

### Legal side of things

From a legal standpoint, we needed to go through the HPE OSRB (Open Source Review Board) to present the project that we wanted to open source. We were asked to follow a process that consisted of four steps:

![HPE OSRB Process](img/wod-osrb.png "HPE OSRB Process")

As the project did not contain any HPE proprietary software, as it is based on open source technologies like Ansible and Jupyter, the process was quite straightforward. Besides, HPE did not want to exploit commercially the generated intellectual property.  We explained to the OSRB that the new architecture of the solution would allow the administrator of the project to separate public content from private content, with private content being a proprietary technology.

This had a huge influence on the future architecture of the project that originally did not allow it. In our case, for instance, any workshop related to an HPE technology like  HPE Ezmeral, would fall into the private part of the project, and therefore, would not appear on the public github repository that we had to create for the overall project distribution.

### Technical side of things

From a technical standpoint, as mentioned above, we had to make sure to separate the public only content from any possible private content. We started by sorting the different workshops (public vs private based). We also had to sort the related scripts that come along with the workshops. Going through this process, we found out that some of the global scripts had to be reworked as well to support any future split of public and private content. Similarly, we had to address any brand specifics, parameterizing instead of hardcoding variables as it was in the first version.

This took us a few months and we are now ready to share with you the result of this work. We will now focus on the architecture side of the Workshops-on-Demand project. 

# The WoD Architecture

 The Workshops-on-Demand concept is fairly simple. The following picture gives you a general idea of how the process works.

![Workshops-on-Demand Concepts High Level View](img/wod-howto.png "Workshops-on-Demand Concepts High Level View")

Now that you understand the basic principle, let's look at the details. The image below shows what happens at each stage from a protocol standpoint.

![Workshops-on-Demand Architecture and Protocols Diagram](img/wod-protocols.png "Workshops-on-Demand Architecture and Protocols Diagram")

## The Register Phase

**1.** Participants start by browsing a frontend web server that presents the catalog of available workshops. They then select one and register for it by entering their email address, first and last names, as well as their company name. Finally, they accept the terms and conditions and hit the register button. 

As the register button is clicked, the frontend server performs a series of actions.

**2.** Assigns a student (i.e student401) from the dedicated workshop range to the participant. Every workshop has a dedicated range of students assigned to it.

Here is a screenshot of the workshop table present in the frontend database server showing API 101 workshops details.

![Workshops Table from API-DB server](img/wod-db-workshop-table.png "Workshops Table from API-DB server")

* Frederic Passeron gets assigned a studentid "student397" for workshop "API101".

  Here are the details of the participant info when registered to a given workshop.

![Customers Table from API-DB server](img/wod-db-customer-table.png "Customers Table from API-DB server")

**3.** An initial email is sent to participants from the API-DB server welcoming them to the workshop and informing them that the deployment is ongoing and that a second email will arrive shortly providing the necessary information required to log onto the workshop environment.

**4.** At the same time, the API-DB server sends the necessary orders through a procmail API call to the backend server. The mail sent to the backend server contains the following details:

* Action Type ( CREATE, CLEANUP, RESET, PURGE)
* Workshop ID
* Student ID

**5.** The backend server recieves the order and processes it by parsing the email recieved using the procmail API. The procmail API automates the management of the workshops.

Like any API, it uses verbs to perform tasks.

* CREATE to deploy a workshop
* CLEANUP to delete a workshop
* RESET to reset associated workshop's resource
* PURGE to delete workshop and reset associated workshop's resource

**CREATE subtasks:**

* **6.** It prepares any infrastructure that might be required for the workshop (Virtual Appliance, Virtual Machine, Docker Container, LDAP config, etc.).
* It generates a random Password for the allocated student.
* It deploys the workshop content on the jupyterhub server in the dedicated student home directory (Notebooks files necessary for the workshops).
* **7.** It sends back the confirmation of the deployment of the workshop, along with the student's required details (i.e password), through API Calls to the API-DB server.

**8.** The frontend server tables will be updated in the following manner:

* The customers table shows an active status for the participant row. The password field has been updated.
* The workshops table also gets updated. The capacity field decrements the number of available seats.
* The students tables gets updated as well by setting the allocated student to active

**9.** The API-DB server sends the second email to each participant providing them with the details to connect to the workshop environment.

## The Run Phase

**10.** From the email, the particpant clicks on the start button. It will open up a browser to the JupyterHub server and directly open the readme first file of the notebook, presenting the workshop's flow.

Participants will go through the different steps and labs of the workshop connecting to the necessary endpoints and leveraging the different kernels available on the JupyterHub server.

Meanwhile, the API-DB server will perform regular checks on how much time has passed. Depending on the time allocation (from 2 to 4 hours) associated with the workshop, the API-DB server will send a reminder email usually an hour before the end of the time allocated. The time count actually starts when participants hit the register for the workshop button. It is mentioned in the terms and conditions.

Finally, when the time is up, the API-DB server sends a new order to the backend to perform either CLEANUP and/or RESET action for the dedicated studentid.

**RESET subtasks:**
[//]: # (TODO: document CLEANUP and PURGE here as well)

* It resets any infrastructure that was required for the workshop (Virtual Appliance, Virtual Machine, Docker Container, LDAP config, etc..).
* It generates a random password for the allocated student.
* It deletes the workshop content on the JupyterHub server in the dedicated student home directory (Notebooks files necessary for the workshop).
* It sends back the confirmation of the CLEANUP or RESET of the workshop along with the student details (i.e password) through API Calls to the API-DB server.

The API-DB server tables will be updated in the following manner:

* The customers table will show an inactive status for the participant row. The password field has been updated. 
* The workshops table gets also updated. The capacity field increment the number of available seats. 
* The students tables gets updated as well by setting the allocated student to inactive.
* The API-DB sends the final email to the participant.

## The React and Reward Phase:

* A final email thanks the students for their participation. It provides a link to an external survey and encourages the participants to share their achievement [badge](ADMIN-GUIDE.md#Badges) on social media like linkedin or twitter.

  Et voila!

# Deploying the infrastructure

The overall infrastructure can run on physical servers or Virtual Machines. We usually designate one server for the frontend, one for the API-DB and PostgreSQL DB and a third (or more) server for the backend.

![WoD Infrastucture](img/wod-infra-open-source.png "WoD Infrastucture")

The project is split into multiple repositories. The project admins will need to decide whether they are willing to develop and propose public-only content to the participants (in which case, the standard environment is sufficient) or add any proprietary and private content (in which case a bit more work is required).

We will start with the simpliest scenario: A public-only approach. Then we will dive into the specificities related to the private approach, which will require the reading of the public-only approach anyway.

## Public-only Deployment: No private backend nor private workshops

**Important Note:**  *This part is compulsory for any type of deployment, public only or public+private.*

### Backend server preparation:

The installation process is handled by a dedicated repo : [wod-install](https://github.com/Workshops-on-Demand/wod-install). This repo needs to be cloned on every single machine  constituting the WoD architecture. Before cloning the `wod-install` repository, you will need to prepare the server that will host the backend features. When ready, you will proceed with the cloning and then the installation process.

#### Prerequesites:

In order to setup the backend server, you will need:

* A fresh minimal OS install on physical/virtualized server running Ubuntu 24.04 leveraging any deployment mechanism of your choice.(e.g. through [iLO](https://en.wikipedia.org/wiki/HPE_Integrated_Lights-Out), automatic installation with [preseed](https://wiki.debian.org/DebianInstaller/Preseed), [virt-lightning](https://github.com/virt-lightning/virt-lightning), etc...).
* A Linux account with sudo priviledges on your Linux distro. Name it `install`   

**Note**: In order to support 100 concurrent users, you need:    

* 2 cpus or more machine
* 128 GB of RAM 
* 500 GB of storage 

We are currently using an either an ProLiant DL360 Gen10 server on our different production sites or QEMU/VMWare VMs on test sites.

When done with OS installation and preparation

* From the WoD-backend server (aka JupyterHub server), as the `install` user, you will need to clone the `wod-install` repo first.

```shellsession
git clone https://github.com/Workshops-on-Demand/wod-install.git
cd wod-install/
```

* Examine default installation parameters and adapt when necessary accordingly. Files are self-documented.

Look at the following file `ansible/group_vars/all.yml`

```shellsession
cat ansible/group_vars/all.yml
---
# We create fixed user accounts to provide an isolated execution environment to run the jupyter notebooks
# They are called studentXXX where XXX is comprised between WODUSERMIN and WODUSERMAX defined below poentially with the addition of an offset (UIDBASE) for their uid/gid
# Their home directory is located under /student and is thus named /student/studentXXX
# Corresponding JupyterHub accounts are also created
#
# WODUSERMIN indicates the starting ID of the Linux and Jupyter user account range
#
WODUSERMIN: 1
#
# WODUSERMAX indicates the ending ID of the Linux and Jupyter user account range
#
WODUSERMAX: 100
#
# Branding management - Use if you want to customize Logo and Notebooks branding
#
BRANDING: "WoD Developer"
BRANDINGWOD: "WoD Developer"
BRANDINGLOGO: "![WoDlogo](img/logo.png)"
BRANDINGLOGOURL: "![WoDlogo](img/logo.png)"
BRANDINGURL: "https://wod.io"
BRANDINGSLACK: ""
BRANDINGX: ""
#
# Survey management - Use if you want to ask for feedbacks on your Workshops - Look at existing conclusion notebooks
SURVEYURL: TBD
SURVEYCHALURL: TBD
#
# You may want to use these variables if you have an OPNSense server as a security FW and allowing http comm internally
#
#OPNSENSEKEY:
#OPNSENSESEC:
#OPNSENSEIP:
#OPNSENSEPORT:
```
See the example below for a backend server.  

### Backend server installation

The installation is based on a common install script [install.sh](https://github.com/Workshops-on-Demand/wod-install/blob/main/install/install.sh) that allows the deployment of the different servers comprising the solution. The script is located under the `wod-install/install/` directory.

It can be called as follows:


`install.sh [-h][-t type][-i ip][-g groupname][-b backend[:beport:[beproto]][-n number][-j backendext[:beportext[:beprotoext]]][-f frontend[:feport[:feproto]]][-w frontendext[:feportext[:feprotoext]]][-a api-db[:apidbport[:apidbproto]]][-e api-dbext[:apidbportext[:apidbprotoext]]][-u user][-p postport][-k][-c][-s sender]`

As you can see on the command line, the -t parameter will define whether you install a backend, an api-db, or a frontend server. Having this information, the script will clone the relevant repositoriesy for the installation. If `t=backend`, then the `wod-backend` repository is cloned as part of the installation process, and the relevant installation scripts are called. Same goes for api-db, and frontend servers.

`-h` provides the help

```shellsession
install@wod-backend2-u24:~/wod-install/install$ sudo ./install.sh -h
install.sh called with -h
install.sh [-h][-t type][-i ip][-g groupname][-b backend[:beport:[beproto]][-n number][-j backendext[:beportext[:beprotoext]]][-f frontend[:feport[:feproto]]][-w frontendext[:feportext[:feprotoext]]][-a api-db[:apidbport[:apidbproto]]][-e api-dbext[:apidbportext[:apidbprotoext]]][-u user][-p postport][-k][-c][-s sender]
 
where:
-a api-db    is the FQDN of the REST API/DB server
             potentially with a port (default 8021)
             potentially with a proto (default http)
             example: api.internal.example.org  
             if empty using the name of the frontend                
 
-b backend   is the FQDN of the backend JupyterHub server,
             potentially with a port (default 8000).
             potentially with a proto (default http)
             if empty uses the local name for the backend
             If you use multiple backend systems corresponding to 
             multiple locations, use option -n to give the backend
             number currently being installed, starting at 1.
 
             When installing the api-db server you have to specify one
             or multiple backend servers, using their FQDN separated 
             with ',' using the same order as given with the -n option
             during backend installation.
 
-e api-dbext is the FQDN of the REST API server accessible externally
             potentially with a port (default 8021)
             potentially with a proto (default http)
             example: api.external.example.org  
             if empty using the name of the api-db                
             useful when the name given with -a doesn't resolve from 
             the client browser
 
-f frontend  is the FQDN of the frontend Web server
             potentially with a port (default 8000).
             potentially with a proto (default http)
             example: fe.external.example.org  
             if empty using the name of the backend                
 
-g groupname is the ansible group_vars name to be used
             example: production, staging, test, ...  
             if empty using 'production'                
 
-i ip        IP address of the backend server being used
             if empty, try to be autodetected from FQDN
             of the backend server
             Used in particular when the IP can't be guessed (Vagrant)
             or when you want to mask the external IP returned
             by an internal one for /etc/hosts creation
 
-j backext   is the FQDN of the backend JupyterHub server accessible externally
             potentially with a port (default 8000).
             potentially with a proto (default http)
             example: jupyterhub.external.example.org  
             if empty using the name of the backend                
             useful when the name given with -b doesn't resolve from 
             the client browser
 
-k           if used, force the re-creation of ssh keys for
             the previously created admin user
             if not used keep the existing keys in place if any
             (backed up and restored)
             if the name of the admin user is changed, new keys 
             systematically re-created
 
-c           if used, force insecured curl communications
             this is particularly useful for self-signed certificate
             on https services
             if not used keep curl verification, preventing self-signed
             certificates to work
 
-n           if used, this indicates the number of the backend 
             currently installed
             used for the backend installation only, when multiple
             backend systems will be used in the configuration
             example (single backend server install on port 9999):
              -b be.int.example.org:9999
             example (first of the 2 backends installed):
              -b be1.int.example.org:8888 -n 1
             example 'second of the 2 backends installed):
              -b be2.int.example.org:8888 -n 2
             example (install of the corresponding api-db server):
              -b be.int.example.org:8888,be2.int.example.org:8888
 
-p postport  is the port on which the postfix service is listening
             on the backend server
             example: -p 10030 
             if empty using default (10025)
 
-s sender    is the e-mail address used in the WoD frontend to send
             API procmail mails to the WoD backend
             example: sender@example.org 
             if empty using wodadmin@localhost
 
-t type      is the installation type
             valid values: appliance, backend, frontend or api-db
             if empty using 'backend'                
 
-u user      is the name of the admin user for the WoD project
             example: mywodadmin 
             if empty using wodadmin               
-w frontext  is the FQDN of the frontend JupyterHub server accessible externally
             potentially with a port (default 8000).
             potentially with a proto (default http)
             example: frontend.external.example.org  
             if empty using the name of the frontend                
             useful to solve CORS errors when external and internal names
             are different
 
 
Full installation example of a stack with:
- 2 backend servers be1 and be2 using port 8010
- 1 api-db server apidb on port 10000 using https
- 1 frontend server front on port 8000
- all declared on the .local network
- internal postfix server running on port 9000
- e-mail sender being wodmailer@local
- ansible groupname being test
- management user being wodmgr
 
On the be1 machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be1.local:8010 -n 1 -t backend
On the be2 machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be2.local:8010 -n 2 -t backend
On the apidb machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -b be1.local:8010,be2.local:8010 -t api-db
On the frontend machine:
  ./install.sh -a apidb.local:10000:https -f front.local:8000 \
  -g test -u wodmgr -p 9000 -s wodmailer@local\
  -t frontend
```

`install.sh` performs the following tasks:

* Calls the `install-system-<< distribution name >>.sh` script which:
  * Updates the Linux distribution package repository.
  * Installs the minimal set of required packages (`ansible, git, jq, openssh server, npm`).
* Creates an admin user as defined upper (default is `wodadmin`) with sudo rights.
* Calls the `install-system-<< type >>.sh` script if it exists (type being one of backend, frontend or api-db.
* Calls the `install-system-common.sh` script that performs the following tasks:
  * Cleanup of previously installed repo (beware if you made local modifications !).
  * Clone required Github repos (leveraging install.repo file) : public Backend and public Private repos.
  * Create ssh keys for `wodadmin` user.
* Calls the `install_system.sh` script that performs the following tasks:    
  * Install the necessary stack based on selected type.
  * Create a `wod.sh` script and wod.yml YAML file in the `/etc` directory containing common variables to be used by all other scripts/playbooks.
  * Source the `/etc/wod.sh` file created.
  * Creates the Ansible inventory files.
  * Creates an Ansible YAML file named after the group name under the private ansible directory in the `generated` sub-directory (allowing private modifications without impacting the upstream repository content).
  * Install required Ansible galaxies (`community.general` and `posix`).
  * Call private scipts during the seyup if they exists (seen later in this guide).
  * Cleanup of previously installed jupyterhub (beware if you made local modifications !).
  * Creates an shell script file named wod-private.sh under the private ansible directory (allowing private modifications without impacting the upstream repository content) containing the credentials for the API and DB access.
  * Calls the Ansible playbook `install_<< type >>.yml` which will perform the software installation and configuration of the WoD << type >> server.
  * Calls the Ansible playbook `check_<< type >>.yml` which will perform the compliance checks (also run every day on the server).

At the end of the installation process:

* you will have a JupyterHub server running on port 8000    
* You will get a new `wodadmin` user (Default admin)    
* You will get a set of 100 students (Default value)    

All playbooks are self-documented. Please check for details.

**Note: A wod-install.log is available under the home folder of the install user under the `.wod-install` directory. It contains the installation log along with a another file containg the wodadmin credentials.**

We leave it to you to handle the necessary port redirection and SSL certificates management when needed. In our case, we went for a simple yet efficient solution based on an OPNSense Firewall along with a HAProxy setup to manage ports'redirection, HTTP to HTTPS Redirection, SSL Certificates. The backend also includes a Fail2ban service for login security management.

At this point, you should be able to access your JupyterHub environment with a few pre-installed set of kernels like `Bash, Python, ansible, ssh, PowerShell`.

You can then start developing new notebooks for your public based environment. Look below in this guide for explanations.

If you need to develop private content that cannot be shared with the wider Open Source Community because of dedicated IP, the next section in this article will explain how to handle this.

## **How to handle private-content based Workshops-on-Demand**

### *(private backend + private workshops on top of default public backend and notebooks)*

The principle remains similar, with a few differences explained below.

* After cloning the wod-install repository, you will fork the following public private [repo](https://github.com/Workshops-on-Demand/wod-private.git) on Github under your own Github account (we will refer to it as `Account`).    
* Next, clone the forked repo.    
* Edit the `all.yml` and `<groupname>` files to customize your setup. This variable `<groupname>` defines possible backend server in your environement. By default, the project comes with a sample working file named `production` in `ansible/group-vars`. But you could have multiple. In our case, we have defined `sandbox`, `test`, `staging` and several `production` files, all defining a different backend environment. These files will be used to override the default values specified by the public version delivered as part of the default public installation.    
* Commit and push changes to your repo.    
* Create an `install.priv` file located in `install` directory when using a private repo (consider looking at [install.repo](https://github.com/Workshops-on-Demand/wod-backend/blob/main/install/install.repo) file for a better understanding of the variables).
* Define the WODPRIVREPO and WODPRIVBRANCH variables as follows:    

  * `WODPRIVBRANCH="main"`     
  * `WODPRIVREPO="git@github.com:Account/Private-Repo.git wod-private"`    

**Note:** When using a token

Please refer to the following [url](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) to generate a `token` file in `install` directory of WoD-backend:

* Edit the `install.priv` file located in `install` directory of wod-install:  

  * Create line before variable declaration: ``token=`cat $EXEPATH/token` ``    
  * Use the token in the url WODPRIVREPO="git clone https://user:$token@github.com/Account/wod-private.git wod-private"    

You are now ready to perform the installation again to support a private repository. 

Please note that this setup phase can be concurrent with the public setup phase. Indeed, the install script should detect the presence of the private repository owing to the presence of the install.priv file. It will automatically adjust the different scripts and variables to add the relevant content. It will actually overload some of the variables with private ones.

You now have a working Workshops-on-Demand backend server in place. Congratulations! The next article in the series will help you better understand the lifecycle of the backend server. How does a workshop registration work from the backend server 's side? How do you manage this server on a daily basis? How and when do you need to update it ? All these questions will be answered in the next article. And from there, we will move to the frontend side of things and finally to a workshop's creation process.

If you need support for this installation process, use our dedicated [slack channel](https://hpedev.slack.com/archives/C01B60X8SSD).

Please be sure to check back [HPE Developer blog site](https://developer.hpe.com/blog) to read all the articles in this series. Also, check out  the Hack Shack for new [workshops](https://developer.hpe.com/hackshack/workshops) [Data Visualization 101](https://developer.hpe.com/hackshack/replays/42) is now available! Stay tuned for additional Workshops-on-Demand in our catalog.In the previous article<https://developer.hpe.com/blog/open-sourcing-workshops-on-demand-part2-deploying-the-backend/> of this series, I explained how to deploy the backend server of our Workshops-on-Demand infrastructure.

In this article, I will dig into details on the backend server. I will cover the inner workings of the registration process, explaining how a workshop is deployed on the backend server. Even though it takes only a few minutes for the backend server to deploy a workshop, there are many processes taking place in the background, which I will cover here.

As a reminder, here is a diagram showing the different parts of the Workshops-on-Demand infrastructure. In this article, I will once again focus on the backend server side and, more precisely, on the JupyterHub server, where all the automation takes place.
[cid:image003.png@01DC3DC6.A29E6220]
Backend server / workshops deployment lifecycle

The following picture depicts what happens on the backend server when a participant registers for a workshop. If you remember from the first article, upon registration the frontend sends instructions to the backend server through a procmail API call so the latter can proceed with the workshop preparation and deployment. Once these tasks are completed, it provides the API-DB server with the relevant information

Let's now look in details what is really happening on the backend server's side:
[cid:image003.png@01DC3DC6.A29E6220]

0- The procmail API: This is a mail parsing process allowing the backend server to retrieve the relevant information in order to perform appropriate actions. As with any API, it uses verbs to perform actions. In our case, we leverage CREATE, CLEANUP, RESET and PURGE.

If you need more info on procmail usage, check this page<https://wiki.archlinux.org/title/Procmail%3E>.

Take a look at the following template of the .procmailrc file that will be expanded at setup time.

MAILDIR=$HOME/.mail      # You'd better make sure it exists

DEFAULT=$MAILDIR/mbox

LOGFILE=$MAILDIR/from



:0b

#* ^From.*{{ WODSENDER }}.*

# \/ defines what will be matched in $MATCH

* ^Subject: *CREATE \/[1-9]+.*

| {{ SCRIPTDIR }}/procmail-action.sh CREATE $MATCH



:0b

#* ^From.*{{ WODSENDER }}.*

# \/ defines what will be matched in $MATCH

* ^Subject: *CLEANUP \/[1-9]+.*

| {{ SCRIPTDIR }}/procmail-action.sh CLEANUP $MATCH



:0b

#* ^From.*{{ WODSENDER }}.*

# \/ defines what will be matched in $MATCH

* ^Subject: *RESET \/[1-9]+.*

| {{ SCRIPTDIR }}/procmail-action.sh RESET $MATCH



:0b

#* ^From.*{{ WODSENDER }}.*

# \/ defines what will be matched in $MATCH

* ^Subject: *PURGE student\/[1-9]+.*

| {{ SCRIPTDIR }}/procmail-action.sh PURGE $MATCH

The From: is important as .procmailrc checks that the sender is the configured one from the frontend server. During the install process, the WODSENDER parameter ID referring to this. Any mail from any other sender but the configured one is not processed.

This API is actually based on a script procmail-action.sh. This script defines the different actions linked to the verbs passed through the API calls via .procmailrc

Let's start with a CREATE scenario looking at the very first lines of the procmail log file.

From xyz@hpe.com<mailto:xyz@hpe.com>  Wed Mar  1 15:10:41 2023

Subject: CREATE 401 825 frederic.passeron@hpe.com<mailto:frederic.passeron@hpe.com>

Folder: /home/wodadmin/wod-backend/scripts/procmail-action.sh CREATE       14

In Subject:, look for the API verb CREATE followed by student id, participant id and finally the registered participant email.

Here are the values respectively:

  *   student id: 401
  *   participant id: 825
  *   participant email: frederic.passeron@hpe.com<mailto:frederic.passeron@hpe.com>

In order to work properly, procmail-action.sh needs to source 3 files:

1- w od.sh

2- r andom.sh

3- f unctions.sh

w od.sh sets a large number of variables: This script is generated at install time as it leverages variables defined at setup time.

# This is the wod.sh script, generated at install

#

# Name of the admin user

export WODUSER=wodadmin



# Name of the WoD machine type (backend, api-db, frontend, appliance)

export WODTYPE=backend



# This main dir is computed and is the backend main dir

export WODBEDIR=/home/wodadmin/wod-backend



# BACKEND PART

# The backend dir has some fixed subdirs

# wod-backend (WODBEDIR)

#    |---------- ansible (ANSIBLEDIR)

#    |---------- scripts (SCRIPTDIR defined in all.yml not here to allow overloading)

#    |---------- sys (SYSDIR)

#    |---------- install

#    |---------- conf

#    |---------- skel

#

export ANSIBLEDIR=$WODBEDIR/ansible

export SYSDIR=$WODBEDIR/sys



# PRIVATE PART

# These 3 dirs have fixed names by default that you can change in this file

# they are placed as sister dirs wrt WODBEDIR

# This is the predefined structure for a private repo

# wod-private (WODPRIVDIR)

#    |---------- ansible (ANSIBLEPRIVDIR)

#    |---------- notebooks (WODPRIVNOBO)

#    |---------- scripts (SCRIPTPRIVDIR)

#

PWODBEDIR=`dirname $WODBEDIR`

export WODPRIVDIR=$PWODBEDIR/wod-private

export ANSIBLEPRIVDIR=$WODPRIVDIR/ansible

export SCRIPTPRIVDIR=$WODPRIVDIR/scripts

export SYSPRIVDIR=$WODPRIVDIR/sys

export WODPRIVNOBO=$WODPRIVDIR/notebooks

WODPRIVINV=""

# Manages private inventory if any

if [ -f $WODPRIVDIR/ansible/inventory ]; then

        WODPRIVINV="-i $WODPRIVDIR/ansible/inventory"

        export WODPRIVINV

fi



# AIP-DB PART

export WODAPIDBDIR=$PWODBEDIR/wod-api-db



# FRONTEND PART

export WODFEDIR=$PWODBEDIR/wod-frontend



# These dirs are also fixed by default and can be changed as needed

export WODNOBO=$PWODBEDIR/wod-notebooks

export STUDDIR=/student

#

export ANSPLAYOPT="-e PBKDIR=staging -e WODUSER=wodadmin -e WODBEDIR=/home/wodadmin/wod-backend -e WODNOBO=/home/wodadmin/wod-notebooks -e WODPRIVNOBO=/home/wodadmin/wod-private/notebooks -e WODPRIVDIR=/home/wodadmin/wod-private -e WODAPIDBDIR=/home/wodadmin/wod-api-db -e WODFEDIR=/home/wodadmin/wod-frontend -e STUDDIR=/student -e ANSIBLEDIR=/home/wodadmin/wod-backend/ansible -e ANSIBLEPRIVDIR=/home/wodadmin/wod-private/ansible -e SCRIPTPRIVDIR=/home/wodadmin/wod-private/scripts -e SYSDIR=/home/wodadmin/wod-backend/sys -e SYSPRIVDIR=/home/wodadmin/wod-private/sys"

export ANSPRIVOPT=" -e @/home/wodadmin/wod-private/ansible/group_vars/all.yml -e @/home/wodadmin/wod-private/ansible/group_vars/staging"

r andom.sh Exports the randomly generated password.

f unctions.sh Is a library of shell functions used by many scripts, among which can be found procmail-action.sh. Details are shown below.

4- procmail-action.sh Calls the necessary functions and scripts to perform the CREATE operation.

5- get_session_token() This function retrieves the necessary token to make an API call to the api-db server.

6- get_workshop_name() This function extracts the workshop name from the mail body. In the body, one will find the workshop name. For example, WKSHP-API101

7- get_workshop_id() From the workshop name, the function get_workshop_id() will get the workshop's ID from the api-db server.

This ID will be used later to get some of the workshop's specifics through additional API calls to the api-db server.

  *   Does the workshop require the use of the student password as a variable?
  *   Does the workshop require LDAP authentication?
  *   Does the workshop require a compiled script?

8- teststdid() This function checks the student ID provided by procmail API is valid: For each workshop, a dedicated student range is allocated. This function exits when the student ID is not in the correct range.

9- generate_randompwd() This function creates a random password for a user. It is used both for local and LDAP users' passwords. If the workshop requires an LDAP authentication (get_ldap_status() functions will return this information) then another function is used to update the LDAP server with the password for the given student (update_ldap_passwd())

The generated password will be sent back to the api-db server so that the frontend server can then send an email to allow participant to connect to the workshop.

10- erase_student() This function erases all the content from the allocated student's home directory. You want to make sure that the home directory is not compromised. You want to start clean.

11- get_compile_status() This function will check if the workshop needs some scripts to be compiled. For instance, if you need to authenticate against a private cloud portal and you don't want your participants to see the credentials, make sure to check the relevant box in the workshop table of the database. This compile feature will compile the authentication scripts into an executable that cannot be edited.

12- If an appliance is needed for the workshop, then the following script is called: create-<WKSHP>.sh .This will prepare the appliance (deploying a docker image on it for instance) and setup user env on the appliance accordingly (ssh keys, skeletons)

For instance, create-WKSHP-ML101.sh will perform the following tasks in order to prepare the appliance for the workshop: It will start by reseting the appliance with the reset-<WKSHP>.sh script. Then, it calls a second script aiming at preparing a generic appliance create-appliance.sh. Once done with these two, it moves on with the proper customization of the appliance for the given student.

See details below.

#!/bin/bash



set -x



source {{ SCRIPTDIR }}/functions.sh



export RTARGET={{ hostvars[inventory_hostname]['IP-WKSHP-ML101'] }}

# Start by cleaning up stuff - do it early as after we setup .ssh content

{{ SCRIPTDIR }}/reset-$ws.sh

{{ SCRIPTDIR }}/create-appliance.sh



NAME=mllab

TMPDIR=/tmp/$NAME.$stdid





mkdir -p $TMPDIR



# Define local variables

echo wid=$wid

APPMIN=`get_range_min $wid`

echo stdid=$stdid

echo APPMIN=$APPMIN

mlport=$(($stdid-$APPMIN+{{ hostvars[inventory_hostname]['MLPORT-WKSHP-ML101'] }}))

mlport2=$(($stdid-$APPMIN+{{ hostvars[inventory_hostname]['MLPORT2-WKSHP-ML101'] }}))

httpport=$(($stdid-$APPMIN+{{ hostvars[inventory_hostname]['HTTPPORT-WKSHP-ML101'] }}))



cat > $TMPDIR/dockerd-entrypoint.sh << EOF

export HTTPPORT

tini -g -- start-notebook.sh &

sleep 3

jupyter lab list | tail -1 | cut -d'=' -f2 | cut -d' ' -f1 > {{ STUDDIR }}/student$stdid/mltoken

sleep infinity

EOF



cat > $TMPDIR/Dockerfile << EOF

FROM ${NAME}:latest

USER root

COPY dockerd-entrypoint.sh /usr/local/bin/

ENTRYPOINT /usr/local/bin/dockerd-entrypoint.sh

RUN mkdir -p {{ STUDDIR }}/student$stdid

RUN useradd student$stdid -u $stdid -g 100 -d {{ STUDDIR }}/student$stdid

RUN chown student$stdid:users {{ STUDDIR }}/student$stdid

# Unlock the account

RUN perl -pi -e "s|^student$stdid:!:|student$stdid:\$6\$rl1WNGdr\$qHyKDW/prwoj5qQckWh13UH3uE9Sp7w43jPzUI9mEV6Y1gZ3MbDDMUX/1sP7ZRnItnGgBEklmsD8vAKgMszkY.:|" /etc/shadow

# In case we need sudo

#RUN echo "student$stdid   ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers

WORKDIR {{ STUDDIR }}/student$stdid

USER student$stdid

ENV NB_USER student$stdid

ENV NB_UID $stdid

ENV HTTPPORT $httpport

RUN git clone https://github.com/snowch/ml-101 {{ STUDDIR }}/student$stdid/

RUN /opt/conda/bin/jupyter-nbconvert --clear-output --inplace {{ STUDDIR }}/student$stdid/*.ipynb

EOF





# Look at https://stackoverflow.com/questions/34264348/docker-inside-docker-container

# and http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/

# For security consider using https://github.com/nestybox/sysbox

cat > $TMPDIR/docker-compose.yml << EOF

version: '3.5'

services:

  $NAME$stdid:

    image: $NAME$stdid

    build: .

    #privileged: true

    ports:

      - "$httpport:8888"

      - "$mlport:4040"

      - "$mlport2:4041"

#    volumes:

#      - /var/run/docker.sock:/var/run/docker.sock

EOF

cat > $TMPDIR/launch-$NAME << EOF

#!/bin/bash

cd $TMPDIR

docker-compose up --build -d

EOF

13- The copy_folder yml playbook is now executed to deploy the notebooks and scripts necessary for the participant to run the workshop. Remember that the participant got a student (with a dedicated student id. For instance: student41) allocated to him/her at the time of the registration. This student id is picked from a range that is allocated for the workshop. The admin decides on the maximum capacity it allocates to a given workshop. c opy_folder.yml: This is historically one of very first playbooks we used and therefore a very important one. It performs the necessary actions to deploy and personnalize (by substituting Ansible variables) the selected notebook to the appropriate student home folder

14- In the certain cases, some post deployment actions are needed. For instance, you may want to git clone some repository to leverage some data stored there. This can only occur when done with the deployment. Therefore, a post-copy-<WKSHP.sh is called.

15- Finally, the workshop is now ready to be used by the participant. The backend therefore, needs to inform the frontend of this. To do so, it will perform two API calls:

  *   The first API call will update the password data for the participant's allocated student.
  *   The second API call will update the participant's allocated student's status to active.

These changes will trigger the frontend web portal application to send a second email to the participant. This email will contain the necessary information for the participant to connect to its notebooks environment. The participant will then run the workshop. For each workshop, a dedicated time window is allocated. Some workshops will take longer to be run than others. The time windows varies from 2 to 4 hours maximum.The system knows how to set it up so that it will time out. This means that once the participant hits the register button on the frontend web portal, the clock starts ticking.

Some background checks take place on the web portal to verify time spent since the registration to a given workshop. As a consequence, a reminder email is sent an hour before the workshop times out. When the bell rings at the end of the class, a new procmail API call is made to the backend server ordering a CLEANUP action. The participant can also trigger this action by registering to a new workshop before the end of the current one. He/She will have to provide the necessary information to the frontend web portal in order to end the current workshop.

Let's see what is happening on the backend server to perform this CLEANUP scenario.
[cid:image003.png@01DC3DC6.A29E6220]

As you can see, it does not differ much from the CREATE. We still need to gather data to interact with the proper workshop from the right student. The .procmail.rc is providing us with this information. Then, the automation kicks in through the procmail-action-sh script.

The verb is now CLEANUP. As a consequence, step 4 is now CLEANUP.

Nothing changes from 5 to 9.

10- get_wod_completion_ratio() Allows us to retrieve information through a simple computing of the numbers of notebooks cells executed thoughout the different exercices of the workshop a ratio. This enables us to see how much of the workshop is actually run. Participants are asked to fill out a form in a conclusion notebook which is present in every student's workshop's folder.

This completion ratio script provides us this data and we store it in our database.

11- API call to send the completion ratio figure to the database. This can be later queried to build up a nice reporting dashboard as explained here by my colleague Didier Lalli, in the following article<https://developer.hpe.com/blog/open-source-elasticsearch-helped-us-globally-support-virtual-labs/>.

12- erase-student(): Now that we have extracted the necessary data from the student's notebooks, we can perform a cleanup of the student folder.

13- cleanup_processes_student(): On top of cleaning up the student folder, we also kill all the allocated student's processes.

14- cleanup-<workshop>.sh: If any appliance is involved, this task will perform the necessary cleanup processes on the appliance.

15- Finally, just like for the creation of a workshop process, we need to tell the frontend that the cleanup is now done. Therefore, several API calls are made to update tables in the database. The new student password is recorded. We also generate a new password at the cleanup phase to prevent unregistered logins. The student status is set to inactive. The capacity figure is incremented by one to make the seat available again.

As for the CREATE phase, the regular checks occuring on the frontent web portal will get these data and trigger the final email to the participant thanking him for his particpation.

Now let's look at the RESET scenario.
[cid:image003.png@01DC3DC6.A29E6220]

You may wonder about the differences between CLEANUP and RESET. CLEANUP only takes care of students whereas RESET takes care of a larger scope. Let me explain...CLEANUP only takes care of student whereas RESET takes care of a larger scope.

When a CLEANUP occurs, it deals with the participant's student workshop and home directory (the workshop directory belonging to the home directory). It cleans up workshop content, ssh keys, skeletons. The RESET will delete leftovers from the workshop's exercices. For instance, when one runs the Kubernetes 101<https://developer.hpe.com/hackshack/workshop/24> workshop, he is creating microservices, he's scaling them, and should at the end of the workshop run some kubectl delete commands to clean up everything. However, some participants may forget to run these clean up steps. And the admin needs to make sure that the next participant who will be assigned the same student environment gets a fresh one. Therefore, some measures have to be taken. These measures take place when a reset flag is associated to the workshop in the database.

During the CLEANUP phase, a check is actually performed to test the presence of this flag through a simple API call on the frontend API-DB server. If the workshop has a reset flag then a dedicated reset-WKSHP.sh script is called and performs the necessary tasks. In the case of Kubernetes 101, it will wipe out any leftovers from the student. In some other cases, it will launch a revert to snapshot script on a virtual machine.

Finally, let's consider the PURGE scenario.
[cid:image003.png@01DC3DC6.A29E6220]

In a perfect world, we would have covered here what one would somehow expect from any kind of API (GET, PUT, DELETE = CREATE, CLEANUP and RESET). But, this is unfortunately not the case. Even though we did our best to harden the deployment automation, failures might occur. Issues could occur at many different levels. From a backhoe loader cutting an internet line on the very morning of a starting event (preventing you from accessing your remote labs) to unplanned power cuts, or misconfigured power redundancy in pdus assignment, there are many examples possible of human factor related issues. As a result, the Jupyterhub server or an appliance might become unreachable, and the automation of the workshop's deployment might fail.

In these very cases, you need to be able to cleanup the mess quickly.
[cid:image003.png@01DC3DC6.A29E6220]

  *   frontend - backend communication issues
  *   JupyterHub server failure
  *   JupyterHub server - appliance server communication issues

The PURGE scenario is therefore triggered on Workshops-on-Demand' deployment failures.

At the registration time, when the participant hits the register button on the frontend web portal, an entry is automatically created in the database for him. It associates the participant to a student and a workshop. It also registers the date and start time of the workshop, sets the participant status to 'welcome' in the database and a first email is sent to the participant from the frontend web portal welcoming him to the Workshop-on-Demand and stating to him that within a few minutes a second email will be sent along with the necessary information (credentials and url) to connect to the workshop's environment.

If for any reason, the deployment of the workshop fails and as a consequence, no API call is made back to the frontend from the backend, the frontend could remain stuck forever and so would the participant. To overcome this, we implemented a check on the frontend web portal to test this welcome status. In a normal scenario, this welcome status gets updated within less than 3 minutes. If the status is not updated within 10 minutes, we consider that something went wrong during the deployment and as a result, a PURGE scenario is initiated to clean up both the backend and the frontend sides of the related registration. Of course, depending of the backend sanity, some actions could also fail. But from our experience, both frontend and backend are really reliable.

Considering now the most common case: the backend server and frontend servers can communicate but the JupyterHub server has issues to communicate with appliances. In terms of tasks associated to the PURGE scenario, you can see that we kept the minimal as there should not be much to clean up on the backend server. Simply consider that it is a CLEANUP scenario without any workshop deployment.

We call the same tasks to begin with as we still need student ID and workshop ID.

We then initiate :

9- generate_randompwd(): We always update the student's password for security reasons.

10- erase-student(): We perform a cleanup of the student folder.

11- API calls to update tables in the database. The new student password is recorded. We also generate a new password at the PURGE phase to prevent unregistered logins. The student status is set to inactive. The capacity figure is incremented by one to make the seat available again.

An email is then sent to the participant explaining to him that we encountered an issue with the deployment and that we apologize for this. The same email is sent to the admin so he can work on the issue.

Now, you should have a clearer view of what is really happening in the background when one registers for a workshop. You can see that I have uncovered many scripts to explain step by step all the stages of a workshop's deployment process. But there is more to be explained. It is obvious that the main function of the backend server is to deploy and run workshops. Nevertheless, as any other server, it cannot live without maintenance.

This subject will be at the core of my next article where I will detail how one needs to manage and work with this server on a daily basis. What we usually call Day 2 operations.

If we can be of any help in clarifying any of this, please reach out to us on Slack<https://slack.hpedev.io/>. Please be sure to drop back at HPE DEV<https://developer.hpe.com/blog> for a follow up on this. Check out also the Hack Shack for new workshops<https://developer.hpe.com/hackshack/workshops>! Willing to collaborate with us? Contact us and let's build together some more workshops! Stay tuned!


Frederic Passeron
Solutions Architect
HPE Developer Community
https://developer.hpe.com/<https://hpe.com/greenlake>
frederic.passeron@hpe.com<mailto:frederic.passeron@hpe.com>

[signature_3167063713]

In previous articles of this series dedicated to the [open sourcing of our Workshops-on-Demand project](https://developer.hpe.com/blog/willing-to-build-up-your-own-workshops-on-demand-infrastructure/), I covered the reasons why we open sourced  the project and how we did it. I also explained in details how you could install your own Workshops-on-Demand backend server. I also took the time to detail the automation that was hosted on this backend server. Today, I plan to describe to you the management of this backend server. This is what is often referred to as Day2 operations. 

Once up and running, the main purpose of the backend server is to deliver workshops-on-Demand. But to do so, it may require updates, upgrades, and/or new kernels for the JupyterHub server. If new workshops are created, this means you'll need new jinja templates for related workshops' scripts (i.e `create<WKSHP>.sh`, `cleanup<WKSHP>.sh`, `reset<WKSHP>.sh`, among others). This also means new variable files. And obviously, these templates and variables will need to be taken into account by scripts and notebooks. Some tasks handle all of this. And that's what I'll show now.

### Backend server management:

If you take a look at the file structure of the `wod-backend` directory, you will discover that the team did its best to sort things properly depending on their relationship to system  or workshops.

#### Content of the backend server:

Simple tree view of the wod-backend directory:

![](img/wod-blogserie2-tree1.png "Tree view of wod-backend directory")

The `ansible` folder contains all the necessary playbooks and variables files to support the main functions of the backend server. It provides playbooks for a minimal installation of the servers or appliances. It also allows the setup of the different types of servers (i.e backend, frontend, and/or api-db), appliances (virtual machines or containers), or workshops as well as maintenance tasks.

At the root of this directory can be found:

`Check*.yml playbooks`: These playbooks are used to perform checks on the different systems. These checks ensure that this a compliant WoD system by checking firewall rules and many other things. You will see this a bit later in more details.

`Copy_folder.yml`: Historically, this is one of very first playbook we used and therefore, it is very important to me. It performs the necessary actions to deploy and personnalize (by substituting Ansible variables) the selected notebook to the appropriate student home folder.

`compile_scripts.yml`: Should you need to hide from the student a simple api call that is made on some private endpoint with non-shareable data (credentials for instance), this playbook will make sure to compile it and create a executable file allowing it to happen. 

`distrib.yml`: This playbook retrieves the distribution name and version from the machine it is run on.

`install_*.yml`: These playbooks take care of installing the necessary packages needed by the defined type (frontend, backend, api-db, base-system or even appliance).

`setup_*.ym`: There are several types of setup playbooks in this directory. 

* `setup_WKSHP-*.yml`: These playbooks are responsible for preparing a base appliance for a given workshop by adding and configuring the necessary packages or services related to the workshop.
* `setup_appliance.yml`: This playbook is used to perform the base setup for a JupyterHub environment server or appliance. It includes setup_base_appliance.yml playbook.
* `setup_base_appliance`: This takes care of setting the minimal requierements for an appliance. It includes `install_base_system.yml` playbook. On top of it, it creates and configures the necessary users.
* `setup_docker_based_appliance.yml`: Quite self explanatory ? it performs setup tasks to enable docker on a given appliance.

It also hosts the `inventory` file describing the role of JupyterHub servers. Place your JupyterHub machine (FQDN) in a group used as PBKDIR namerole.

```shellsession
#
# Place to your JupyterHub machine (FQDN) in a group used as PBKDIR name.
#
[production]
127.0.0.1  ansible_connection=localhost
```

The `conf` folder hosts configuration files in a Jinja format. Once expanded, the resulting files will be used by relevant workshops. I will explain in a future article all the steps and requirements to create a workshop.

As part of the refactoring work to open source the project, we reaaranged the different scripts' locations. We have created an install folder to handle the different installation scripts either from a JupyterHub's perpective or from an appliance's standpoint, too.

We separated the workshops' related scripts from the system ones. When one creates a workshop, one needs to provide a series of notebooks and in some cases some scripts to manage the creation and setup of a related appliance along with additional scripts to manage its lifecycle in the overall Workshops-on-Demand architecture (Create, Cleanup, Reset scripts at deployment or Cleanup times). These scripts need to be located in the `scripts` folder. On the other hand, the system scripts are located in the `sys` folder.

![](img/tree-wkshop2.png "Tree view of the sys directory")

This directory hosts important configuration files for both the system and JupyterHub. You can see for instance `fail2ban` configuration files. Some Jinja templates are present here, too. These templates will be expanded through the `deliver` mechanism allowing the creation of files customized with Ansible variables. All the WoD related tasks are prefixed with WoD for better understanding and ease of use.

These Jinja templates can refer to some JupyterHub kernel needs like `wod-build-evcxr.sh.j2` that aims at creating a script allowing the rust kernel installation. Some other templates are related to the system and JupyterHub. `wod-kill-processes.pl.j2` has been created after discovering the harsh reality of online mining. In a ideal world, I would not have to explain further as the script would not be needed. Unfortunately, this is not the case. When one offers access to some hardware freely online, sooner or later, he can expect to  see his original idea to be hyjacked.

Let's say that you want to provide some AI/ML 101 type of workshops. As part of it, 
you may consider providing servers with some GPUs. Any twisted minded cryptominer discovering your resources will definitely think he's hits the jackpot! This little anecdot actually happened to us and not only on GPU based servers, some regular servers got hit as well. We found out that performance on some servers became very poor and when looking into it, we found some scripts that were not supposed to run there. As a result, we implemented monitors to check the load on our servers and made sure that to  kill any suspicious processes before kicking out the misbehaving student.

`wod-test-action.sh.j2` is another interesting template that will create a script that can be used for testing workshops. This script mimics the procmail API and actually enables you to test the complete lifecycle of a workshop from deployment to cleanup or reset.

```shellsession
wodadmin@server:/usr/local/bin$ ./wod-test-action.sh
Syntax: wod-test-action.sh <CREATE|CLEANUP|RESET|PURGE|PDF|WORD> WKSHOP [MIN[,MAX]
ACTION is mandatory
wodadmin@server:/usr/local/bin$
```

It requires the verb, the workshop's name and the student id. Using the script, one does not need to provide participant id.  The script is run locally on the JupyterHub server.

```shellsession
wodadmin@server:/usr/local/bin$ ./wod-test-action.sh
Syntax: wod-test-action.sh <CREATE|CLEANUP|RESET|PURGE|PDF|WORD> WKSHOP [MIN[,MAX]
ACTION is mandatory
wodadmin@server:/usr/local/bin$ ./wod-test-action.sh CREATE WKSHP-API101 121
Action: CREATE
We are working on WKSHP-API101
Student range: 121
Sending a mail to CREATE student 121 for workshop WKSHP-API101
220 server.xyz.com ESMTP Postfix (Ubuntu)
250 2.1.0 Ok
250 2.1.5 Ok
354 End data with <CR><LF>.<CR><LF>
250 2.0.0 Ok: queued as 9749E15403AB
221 2.0.0 Bye
```

In order to retrieve the result of the script, you simply need to run a `tail` command.

```shellsession
wodadmin@server:~$ tail -100f .mail/from
++ date
....
>From xyz@hpe.com  Fri Mar  3 09:08:35 2023
 Subject: CREATE 121 0
  Folder: /home/wodadmin/wod-backend/scripts/procmail-action.sh CREATE       11
+ source /home/wodadmin/wod-backend/scripts/wod.sh
....
+ echo 'end of procmail-action for student 121 (passwd werty123) with workshop WKSHP-API101 with action CREATE at Fri Mar  3 09:11:39 UTC 2023'
```

The very last line of the trace will provide you with the credentials necessary to test your workshop. 

There are two types of activities that can occur on the backend server: punctual or regular. The punctual activity is one that is performed once every now and then. The regular one is usually set up on the backend server as a cron job. Sometimes however, one of these cron tasks can be forced manually if necessary. One of the most important scheduled task is the `deliver` task. I will explain it later on in this chapter. I will start now by explaining an important possible punctual task, the update of the backend server.

### Update of the backend server:

The backend server hosts all the necessary content for delivering workshops: it supplies notebooks,scripts and playbooks to deploy and personalize them. It also hosts some services that are needed by the overall architecture solution (JupyterHub, Procmail, Fail2ban among others).

Services are installed once and for all at the installation time. These services may evolve over time. One may need to update the JupyterHub application to fix a bug or get new features. In the same fashion, you may consider bumping from one Python version to a new major one. If you are willing to update these services or add new ones, you will need to update the relevant installation playbooks in `wod-backend/ansible` directory.

Here is a small extract of the `install_backend.yml` playbook: Full version [here](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/install_backend.yml)

```shellsession
vi install_backend
- hosts: all
  gather_facts: true
  vars:
    IJAVAVER: "1.3.0"
    KUBECTLVER: "1.21.6"

  tasks:
    - name: Include variables for the underlying distribution
      include_vars: "{{ ANSIBLEDIR }}/group_vars/{{ ansible_distribution }}-{{ ansible_distribution_major_version }}.yml"

    - name: Base setup for a JupyterHub environment server or appliance
      include_tasks: "{{ ANSIBLEDIR }}/setup_base_appliance.yml"

    - name: Add CentOS SC repository into repo list
      become: yes
      become_user: root
      yum:
        name: centos-release-scl-rh
        state: present
      when:
        - ansible_distribution == "CentOS"
        - ansible_distribution_major_version >= "7"

    - name: Add conda GPG Key to APT
      become: yes
      become_user: root
      apt_key:
        url: https://repo.anaconda.com/pkgs/misc/gpgkeys/anaconda.asc
        state: present
      when:
       - ansible_distribution == "Ubuntu"
       - ansible_distribution_major_version >= "20"

      # TODO: Do it for EPEL if really needed
    - name: Add conda APT repository
      become: yes
      become_user: root
      apt_repository:
        repo: deb [arch=amd64] https://repo.anaconda.com/pkgs/misc/debrepo/conda stable main
        state: present
      when:
       - ansible_distribution == "Ubuntu"
```

Possible Use Cases:

* Upgrade to a newer version of JupyterHub
* Add a new kernel to JupyterHub
* Add a new Ansible Galaxy collection
* Add a new PowerShell library 
* Add a new package needed by a workshop. 

For e.g:

* Kubectl client  
* Terraform client
* PowerShell module  
* Python Library

You will start by moving to your public backend forked repository and apply the necessary changes before committing and push locally. 

Then you will perform a merge request with the main repository. We plan to integrate here  in a proper CICD (continuous integration continous development) pipeline to allow a vagrant based test deployment. Whenever someone performs a merge request on the main repo, the test deployment task kicks in and deploys a virtual backend server on which the new version of the installation process is automatically tested. When successful, the merge request is accepted. Once merged, you will need to move to your backend server and perform git remote update and git rebase on the wod-backend directory. Once done, you will then be able to perform the installation process.

### Regular maintenance of the backend server:

On a daily basis, some tasks are launched to check the integrity of the backend server. Some tasks are related to the security integrity of the system. The following playbook is at the heart of this verification: **wod-backend/ansible/check_backend.yml**. Full version of the file is available [here](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_backend.yml) for review.

It checks a quite long list of items like:

* Wod System compliancy: is this really a WoD system? by calling out [check_system.yml](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_system.yml) playbook. 

This first check includes:  

* nproc hard and soft limits 
* nofile hard and soft limits
* Setup sysctl params 

  * net.ipv4.tcp_keepalive_time, value: "1800"  
  * kernel.threads-max, value: "4096000"  
  * kernel.pid_max, value: "200000" 
  * vm.max_map_count, value: "600000"
  * Setup UDP and TCP firewall rules
* Enable services:

  * Firewalld 
  * Ntp 
* Student Management:

  * Ensure limits are correct for students accounts 
  * Copy the skeleton content under /etc/skel
  * Test `.profile` file
  * Ensure vim is the default EDITOR
  * Setup `logind.conf`
  * Manage `/etc/hosts` file
  * Install the pkg update script
  * Setup `crontab` for daily pkg security update
  * Deliver create/reset/setup scripts as ansible template for variable expansion
  * Install utility scripts
  * Deliver the system scripts (`cleanup-processes.sh.j2`)
  * Installation of the cleanup-processes script
  * Setup weekly cleanup processes task
  * Enable WoD service
  * Test private tasks YAML file
  * Call private tasks if available. It performs the private part before users management to allow interruption of the deliver script during normal operations - waiting till end of users management can take hours for 2000 users. Potential impact: private scripts are run before users creation, so may miss some part of setup.
* User Management:

  * Remove existing JupyterHub users
  * Remove Linux users and their home directory
  * Ensure dedicated students groups exist
  * Ensure Linux students users exists with their home directory
  * Ensure JupyterHub students users exist
  * Setup ACL for students with JupyterHub account
  * Setup default ACL for students with JupyterHub account

A similar set of scripts exist for the different parts of the solution ([check_api-db.yml](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_api-db.yml) for api-db server, [check_frontend.yml ](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/check_frontend.yml)for frontend server for instance). 

You should now have a better understanding of the maintenance tasks associated to the backend server. Similar actions are available for the other components of the project. Checking tasks have been created for the frontend and api-db server. Having now mostly covered all the subjects related to the backend server from an infrastructure standpoint, it is high time to discuss the content part. In my next blog, I plan to describe the workshop creation process.  Time to understand how to build up some content for the JupyterHub server!

If we can be of any help in clarifying any of this, please reach out to us on [Slack](https://slack.hpedev.io/). Please be sure to check back at [HPE DEV](https://developer.hpe.com/blog) for a follow up on this. Also, don't forget to check out also the Hack Shack for new [workshops](https://developer.hpe.com/hackshack/workshops)! Willing to collaborate with us? Contact us so we can build more workshops!
In this article that is part of our series dedicated on [open sourcing of our Workshops-on-Demand project](https://developer.hpe.com/blog/willing-to-build-up-your-own-workshops-on-demand-infrastructure/), I will focus on the steps necessary  to build up a new workshop. In my previous posts, I have already covered most of the pieces on how to set up the infrastructure to support the workshops. Now let's focus a little more on the content creation.

# Overview

Let's start with a simple flowchart describing the 10000-foot view of the creation process:

![](img/wod-b-process.png "Workshop's creation flow.")

As you can see, there's no rocket science here. Just common sense. Depending on the workshop you wish to create, some obvious requirements should show up. A workshop based on a programmatic language, for instance, may require the relevant kernel to be set up on the JupyterHub server. The following [page](https://gist.github.com/chronitis/682c4e0d9f663e85e3d87e97cd7d1624) lists all available kernels.

Some workshops might need a specific infrastructure set up in order to run. A Kubernetes 101 workshop, for instance, could not exist without the presence of a proper Kubernetes cluster. The same thing goes for any HPE-related solutions.

In setting up the infrastructure, there are a number of things you need at a minimum. The design of this open-sourcing project makes it very easy to deploy a development and test environment, a staging environment, and at least one production environment. 

Let's look at how each of these environments is defined :

1. **Development Environment:** This is where application/system development tasks, such as designing, programming, debugging of a workshop, etc., take place.
2. **Test Environment:** As the name implies, this is where the workshop testing is conducted to find and fix errors.
3. **Staging Environment:** Here, all the work done in the development environment is merged into the built system (often used to automate the process of software compilation) before it is moved into the production environment.
4. **Production Environment:** The last environment in workshop development, this is where new builds/updates are moved into production for end users.

When the HPE Developer Community began implementing their Workshop-on-Demand program, they originally only had a development and a test & staging environment on one end and a production environment on the other.

In this post, I won't focus on the subject selection process. I'll leave that to you to figure it out. I will, however, talk a little bit again about the infrastructure, especially the dedicated scripts and variables that you need to create to support the lifecycle of the workshop. As usual, there are two sides to the workshop's creation--what should be done on the backend and what needs to be done mainly for the api db server.

![](img/wod-blogserie3-archi3.png "WOD Overview.")

## What is a workshop? What do you need to develop?

Let me use an example to better explain this. There's this engineer, Matt, who has a great deal of knowledge that he would like to share. He was kind enough to agree to working with me on creating a new workshop. After our first meeting, where I explained the creation process and the expectations, we were able to quickly start designing a workshop that would help him do this.

We defined what was needed:

* A set of notebooks that will be used by the student
* Containing instructions cells in markdown and run code cells leveraging the relevant kernel. If you are not familiar with Jupyter notebooks, a simple [101 workshop](https://developer.hpe.com/hackshack/workshop/25) is available in our Workshops-on-Demand 's catalog.
* A student range for the workshop.

A workshop should contain at least:

* 0-ReadMeFirst.ipynb 
* 1-WKSHP-LAB1.ipynb
* 2-WKSHP-LAB2.ipynb
* 3-WKSHP-Conclusion.ipynb
* LICENCE.MD
* A pictures folder (if any screenshot is required in lab instructions)
* A README.md (0-ReadMeFirst.ipynb in md format)
* A wod.yml

To make the workshop compliant to our platform, Matt just needs to provide a final file that contains a set of metadata that will be used  for the workshop's integration into the infrastructure. this file is called **wod.yml**.

Matt can leverage a simple **wod.yml** file containing them and that can be later parsed in order to feed the database with the relevant info. Quite handy, no? Moreover, the same script that will create the workshop entry in the database can also be used to update it.

Here is an example of such a file:

```yaml
%YAML 1.1
# Meta data for the GO101 Workshop to populate seeder
---
name: 'GO 101 - A simple introduction to Go Programming Language'
description: 'Go, also called Golang or Go language, is an open source programming language that Google developed. Software developers use Go in an array of operating systems and frameworks to develop web applications, cloud and networking services, and other types of software. This workshop will drive you through the basics of this programming language.'
active: true
capacity: 20
priority: 1
range: [151-170]
reset: false
ldap: false
location: 'fully qualified domain name of default JupyterHub server'
replayId: 31
varpass: false
compile: false
workshopImg: 'https://us-central1-grommet-designer.cloudfunctions.net/images/frederic-passeron-hpe-com/WOD-GO-101-A-simp-introduction-to-Go-programming-language.jpeg'
badgeImg: 'https://us-central1-grommet-designer.cloudfunctions.net/images/frederic-passeron-hpe-com/go101-a-simple-introduction-to-go-programming-language.jpg'
beta: false
category: ['Open Source']
duration: 4
alternateLocation: ['fully qualified domain name of an alternate JupyterHub server']
presenter: 'Matthew Doddler'
role: 'FullStack developer'
avatar: 'img/SpeakerImages/MattD.jpg'
replayLink: 'https://hpe-developer-portal.s3.amazonaws.com/Workshops-on-Demand-Coming-Soon-Replay.mp4'
```

The following file will be used to update the **workshops table** in the database. Let's have a look at what a new entry could look like:

![](img/wod-db-go-1.png "GO 101 Workshop DB screenshot")

![](img/wod-db-go-2.png "GO 101 Workshop DB screenshot")

As a contributor, Matt should be able to provide all the following details.

* **ID:** A workshop ID to be used by backend server automation and Replays table to reference the associated replay video of the workshop (automatically created at the import of the wod.yml file process)
* **name:** The workshop's name as it will will be displayed on the registration portal
* **notebook:** The name of the folder containing all the workshop's notebooks (automatically created at the import of the wod.yml file process)
* **description:** The workshop's abstract as it will will be displayed on the registration portal
* **avatar, role and replayLink** are superseded by entries in the replay table (I will explain later)
* **replayId:** This entry links the dedicated replay video to the workshop and enables the presence of the replay in the learn more page of the workshop (automatically created at the import of the wod.yml file process)
* **category:** The workshops' registration portal proposes several filters to display the catlog's content. You can view all workshops, the most poular ones, or by category. Use this field to sort workshops accordingly.
* **duration:** All workshops are time bombed. You will define here the time allocated to perform the workshop
* **active:** Tag to set to enable visibility of the workshop's tile in the registration portal
* **workshopImg:** As part of the lifecycle of the workshop, several emails are sent to the student. A workshop image is embedded in the first emails
* **session type:** Workshops-on-Demand by default (automatically created at the import of the wod.yml file process)

The following fields are required by the infrastructure. In this example, I will work as the infrastructure Admin with Matt to define them.

* **capacity:** The number of maximum concurrent students allowed to take on the workshop
* **range:** The range between which students get picked at registration time
* **reset and ldap** entries are to be used by backend server automation if dedicated reset scripts and ldap authentication are required by the workshop
* **location:** If your setup includes multiple JupyterHub servers, use this field to allocate workshops according to your needs.
* **compile:** This entry will be filled with the name of a script to be compiled at deployment time. This feature allows the admin to hide login scripts and credentials in non-editable executable files.
* **varpass:**  This defines whether or not a workshop requires a password variable needs to be leveraged
* **badgeImg:** As part of the lifecycle of the workshop, several emails are sent to the student. In the final email, a badge is included. It allows the student to share its accomplishment on social media like linkedin for instance.
* **beta:** Not implemented yet :-)
* **alternateLocation:** (Future development) The purpose is to allow automation of the relocation of a workshop in case of primary location's failure
* **replayLink:** YouTube link of the recorded video to be used as a replay
* **replayid:** This ID is used to link the correct video to the workshop. This is the replayId present in the workshops table (automatically created at the import of the wod.yml file process)
* **monoAppliance:** Some workshops require a single dedicated appliance
* **multiAppliance:** Some workshops require multiple dedicated appliances

***Note:*** Both WorkshopImg and BadgeImg are delivered by the frontend web server. 

If you feel you need more details about the registration process, please take a look at the **Register Phase** paragraph in [the following introductionary blog](https://developer.hpe.com/blog/willing-to-build-up-your-own-workshops-on-demand-infrastructure/).

Matt will create a simple workshop that does not require any infrastructure but the JupyterHub itself. As far as the infrastructure's requirements, a new kernel was needed. No additional scripts were required for this workshop.

As an admin of the Workshops-on-Demand infrastructure, I have to perform several tasks on a development environment and a staging environment:

### On the backend server:

Testing and validating installation of the new kernel on the staging backend server by:

1. Creating a new branch for this test
2. Modifying the [backend server installation yaml file ](https://github.com/Workshops-on-Demand/wod-backend/blob/main/ansible/install_backend.yml#L326)to include the new kernel
3. Validating the changes by testing a new backend install process

Pushing the changes to the GitHub repo:

1. Create a user for the workshop developer on the test/dev and staging backend servers
2. Provide to the developer the necessary information to connect to the test/dev and staging backend servers
3. Copy in the developer's home folder a workshop template containing examples of introduction, conclusion, and lab notebooks, allowing him to start his work
4. Give the developer the wod-notebook repo url for him to fork the repo and work locally on his machine (when the workshop does not require an appliance but just a Jupyter kernel for instance)
5. When ready, a pull request can be made. The admin can then review and accept it. The admin can then perform the necessary steps required to prepare the infrastructure to host the workshop

### On the api-db server:

Connecting to the api-db server as wodadmin user:

1. Switch to the relevant branch for the new workshop and perform a git remote update / rebase in the relevant notebook directory.
2. Move to wod-api-db/scripts directory
3. Update the database by running the wod-update-db.sh script.

This script will update the Database workshops table with the new workshop's entry.

As the developer of the Workshops-on-Demand content, Matt had to perform several tasks:

### On the backend server:

1. Log on to the backend server and clone the notebook repo in his home folder, or as explained earlier, fork the repo on his laptop and work from there
2. Create a new branch for his workshop following the naming convention defined with the admin
3. Leverage the template provided by me to build up the content of his workshop
4. Test the workshop locally on his laptop or on the dev server leveraging the `wod-test-action.sh` script
5. Test the workshop using the staging registration portal
6. When all tests are green, create a pull request to merge content with the master repo

As an admin, I would need to check the pull request and accept it. Once done, the test/dev and staging environments will require an update.

1. Log on to the backend server as wodadmin and update the notebook repository.
2. Run a wod-deliver to update the relevant backend server.

   ```
   git remote update
   git rebase
   wod-deliver
   ```

The very same processes will apply to the move to production phase.

# Complex workshop example:

If you've ever had to develop a workshop for something you're not as familiar with, you'll want to meet with SMEs and explain what your goals are and how to achieve them. Once you have a clearer understanding of the technology involved, you can move on to determine what the best platform would be on which to run the workshop. I can give you an example here focusing on Ansible, an automation tool. I was not originally familiar with it before I developed the Ansible 101 Workshop. In the steps below, I'll explain how I pulled it together..

As the admin for the HPE Developer Community's Workshops-on-Demand infrastructure, I had to perform several tasks in order to set up this workshop. I had to determine what the workshop required from a backend and fully test it.

##### On the backend server:

The  workshop will require:

* A dedicated server running Linux to allow the student to run Ansible playbooks against during the workshops' labs.

  * This means preparing a server (VM or physical) : We will consider it as an appliance
  * Updating the relevant variable file to associate the IP address of the server to the workshop (there could be multiple servers too associated to a given workshop)
* A set of scripts under wod-backend/scripts or wod-private/scripts folders depending on the nature of the workshop to manage the workshop's lifecycle

  * Some generic scripts applying to all workshops' appliances : general setup phase setting up common requirements for any appliance (student users creation, ssh keys, etc..) up to some specific ones dedicated to a given workshop

    * create-appliance.sh (ssh keys, ldap setup)
    * setup-appliance.sh \[WKSHP-NAME] (Student setup, Appliance Setup, Workshop setup) calls:

      * setup-globalappliance.sh (global / generic setup)
      * setup-\[WKSHP-NAME].sh (Prepare appliance with workshop's reqs, Docker image for instance)
    * create-\[WKSHP-NAME].sh (called at deployement time to instantiate the necessary appliance(s) requiered by the workshop 
    * reset-appliance (reset ssh keys and students credentials on appliance
    * cleanup-\[WKSHP-NAME].sh (takes care of cleanup some workshop's specifics)
    * reset-\[WKSHP-NAME].sh (reset of the workshop's appliance, docker compose down of a container for instance)
* A set of variables to be leveraged by the notebooks. These variables are to be set in yml format. They will be parsed at deployment time to set student ids, appliance IP addresses, and other relevant parameters like ports, or simulated hardware information

When all the scripts are functional and the necessary actions have been performed both on backend and frontend servers, some functional tests can be conducted using cli and later webui as described earlier for the simple workshop example.

Testing the workshop: 

* One can leverage the wod-test-action.sh script to test a workshop lifecycle action from deployment (CREATE) to CLEANUP, RESET, or PURGE.

  ```shell
  dev@dev3:~$ wod-test-action.sh
  Syntax: wod-test-action.sh <CREATE|CLEANUP|RESET|PURGE|PDF|WORD> WKSHOP [MIN[,MAX]
  ACTION is mandatory
  ```

`Note: The available trace under ~/.mail/from will detail the different steps of the action and allow you to troubelshoot any issue.`

When all tests have validated the workshop, it can follow the move to prod cycle.

You should now have a better understanding of the necessary tasks associated to the creation of a workshop. As you can see, it requires steps on the various sides of the infrastructure.

This was the last blog post of the series. The Workshops-on-Demand project is available [here](https://github.com/Workshops-on-Demand). Further update of the documentation will occur on this github repository.

If we can be of any help in clarifying any of this, please reach out to us on [Slack](https://developer.hpe.com/slack-signup/). Please be sure to check back at [HPE DEV](https://developer.hpe.com/blog) for a follow up on this. Also, don't forget to check out also the Hack Shack for new [workshops](https://developer.hpe.com/hackshack/workshops)! Willing to collaborate with us? Contact us so we can build more workshops!
