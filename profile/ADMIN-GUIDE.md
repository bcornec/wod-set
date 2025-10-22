![Frontend](img/wod-frontend.img)

Leveraging the JupyterHub technology, the HPE DEV team, at the origin of the project, created a set of hands-on technology training workshops where anyone could connect to our service and benefit from notebook-based workshops at any time, and from anywhere. Read our [User's Guide](USER-GUIDE.md) to see it in action.

Now, let me explain how we made this possible…

## Where it all comes from

We built up our first JupyterHub server for the virtual HPE Technology & Solutions Summit (TSS) that took place in March 2020 and used it to deliver the workshops we would normally give in person in a virtual way. We hosted just a few Jupyter notebooks-based workshops and found that they scored really well in feedback we received from the students. All notebooks were delivered through a single instance of a JupyterHub server.

During the event, we ran only a single workshop at a time. We first set up a range of users (students) to match a given workshop. It required a lot of manual setup, a few scripts and a single Ansible playbook to handle the copy of the workshop content assigned to the desired range of students, with a few variable substitutions like student IDs, passwords, and some API endpoints when relevant. The student assignment was done manually, at the beginning of every workshop. For instance, Fred was student1, Didier was student2, and so on… which was quite cumbersome. When you only have a range of 25 students, one or two people handling the back end is sufficient. But if 40 people show up, then it becomes tricky.

Of course, as automation minded people, we could not stay like that for long, so we decided to build a framework to help us deliver more Workshops on Demand (aka WoD) and began our 5 years journey of developing it :-)

## Architecture considerations

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
![Multiple servers/services](img/wod-infra-open-source.png)
- A frontend server to manage he workshop list and allow user registration
- An API server coupled with the Database to orchestrate the platform
- A backend server on which will run the JupyterHub service and where the Notebooks will be deployed
- Appliances servers running the technology at work in the Notebook (optional)

For our automation, we have playbooks that would perform:
- the server configuration (based on an existing deployed Operating System with just git installed) including:
    - Repository Update
    - Apps Installation
    - System Performance Tuning including kernels setup & configuration 
    and depending on the server:
    - on backend:
        - JupyterHub application installation and configuration
        - Linux students creation
        - JupyterHub users creation
    - on frontend:
         - Javascript setup with npm
         - portal installation and launch
    - on the API/DB:
         - Javascript setup with npm
         - PostgreSQL install and setup with sequelize
         - REST API server installation and launch
- the server conformity management (run after the previous playbook and on demand and nightly to ensure that the configuration is consistent and up to date):
    - System Update
    - Security Setup
    - Services Conformity and check
    - Templating of configration and data files
- the notebooks deployment on the JupyterHub server:
    - Templating the Notebook and data files
    - Security setup

We created as many Git repositories as needed, for the infrastructure management:
![WoD repositories](img/wod-repositories.png)
- [wod-frontend](https://github.com/Workshops-on-Demand/wod-frontend) for the portal
- [wod-api-db](https://github.com/Workshops-on-Demand/wod-api-db) for the REST API and PostgreSQL DB
- [wod-backend](https://github.com/Workshops-on-Demand/wod-backend) for the JupyterHub
- [wod-notebooks](https://github.com/Workshops-on-Demand/wod-notebooks) for the Notebooks provided
- [wod-private](https://github.com/Workshops-on-Demand/wod-private) as a template fo using a private setup alongside the public one.
- [wod-install](https://github.com/Workshops-on-Demand/wod-install) for the installation of the infrastructure
- [wod-doc](https://github.com/Workshops-on-Demand/.github) for the project documentation

## How it works

Let me first show you how the overall process works for our Workshops-on-Demand:

![Workshops-on-Demand](https://hpe-developer-portal.s3.amazonaws.com/uploads/media/2021/4/hackshack2-1617960613898.png)

If you’re looking for a live explanation of this automation, you can review [the following session](https://www.youtube.com/watch?v=D6Ss3T2p008&t=515s) Bruno Cornec and I delivered at Linuxconf in Australia in January 2021.



**Step 1:** The customer registers for a workshop online at our virtual [Hack Shack](/hackshack/workshops). When clicking the register button for the selected workshop and after agreeing to the terms and conditions, the front-end triggers the first REST API calls to:

-    Register the student in the Customers Database.
-    Send a welcome email to the student
-    Assign a student ID to him/her according to the student range allocated to the given workshop 

**Step 2:** The registration App then orders (through a mail API call managed by procmail) the backend to:

-    Generate a random password for the selected student
-    Deploy the selected workshop

**Step 3:** The backend Infrastructure calls back the registration application using its REST API to:

-    Provide back the new student password
-    Make the student’s database record active
 -    Decrement Workshop capacity

**Step 4:** The registration App sends:
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



The registration app actually sends an email to the backend with a dedicated format (an API!) that is then parsed by the Procmail process to retrieve the necessary information to perform the different tasks. We use 3 verbs: CREATE (to setup to the student environment as described upper), DELETE (to remove the user from tables), and RESET (to clean up the student content and reset the back-end infrastructure when needed).

At the end of this automated process, the backend makes a series of API calls to the registration app to send back the required information, like a new password, workshop status, etc.



More info on Procmail can be found [here](https://en.wikipedia.org/wiki/Procmail).



## Flying

In the fall, we started a pilot phase for a month. We opened the platform internally and gathered some feedback. We managed to discover a few bugs then that are now corrected. The platform went live in November 2020 and is used on a daily basis.

Since then, we have added new workshops on a monthly basis.  We lately returned to TSS using the Workshops-on-Demand to deliver our content; this time without any manual process in the scheme.



Our next steps include the following:



-   Implement complete deployment scenario
    -   OS Bare metal deployment
-    Provide a new backend on HPE GreenLake
     -     We have now a JupyterHub server running in HPE GreenLake
     -    We validated the relationship between this new backend and our registration application
     -   More testing to come
-    Automate the student ID range booking using a YAML configuration file
-    Open source our Workshops-on-Demand Notebooks content:
    -    This is work in progress. The repository is ready. Licensing is ok. 
    -    Only missing some automation layer again to sync different repositories

Before leaving you with some final thoughts, let me show you a screenshot of our dashboard presenting:


-    The number of Active Workshops
-    The active workshops by type
-    The total number of registrations from November 1st 2020 till today
-    The total number of Customer (student) registrations
-    The total workshops split


![Dashboard](https://hpe-developer-portal.s3.amazonaws.com/uploads/media/2021/4/hackshack3-1617960606167.png)



Didier Lalli from our HPE DEV team created this informative dashboard and wrote a blog about it. If you are interested in learning more about ElasticSearch, read it out [here](https://developer.hpe.com/blog/open-source-elasticsearch-helped-us-globally-support-virtual-labs).



As you can see, we have made significant progress over the past year, moving from a heavily manually oriented approach to now a fully automated one. If you are interested in seeing the result for yourself, [please register for one of our Workshops-on-Demand](/hackshack/workshops). Don’t forget to fill out the survey at the end of the workshop to provide us with feedback on your experience, as well as subjects you would like to see covered in the near future, as this really helps to improve the program.
