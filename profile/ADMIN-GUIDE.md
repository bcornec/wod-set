![Frontend](img/wod-frontend.png)

Leveraging the JupyterHub technology, the HPE DEV team, at the origin of the project, created a set of hands-on technology training workshops where anyone could connect to our service and benefit from notebook-based workshops at any time, and from anywhere. Read our [User's Guide](USER-GUIDE.md) to see it in action.

Now, let me explain how we made this possible…

## Where it all comes from

We built up our first JupyterHub server for the virtual HPE Technology & Solutions Summit (TSS) that took place in March 2020 and used it to deliver the workshops we would normally give in person in a virtual way. We hosted just a few Jupyter notebooks-based workshops and found that they scored really well in feedback we received from the students. All notebooks were delivered through a single instance of a JupyterHub server.

During the event, we ran only a single workshop at a time. We first set up a range of users (students) to match a given workshop. It required a lot of manual setup, a few scripts and a single Ansible playbook to handle the copy of the workshop content assigned to the desired range of students, with a few variable substitutions like student IDs, passwords, and some API endpoints when relevant. The student assignment was done manually, at the beginning of every workshop. For instance, Fred was student1, Didier was student2, and so on… which was quite cumbersome. When you only have a range of 25 students, one or two people handling the back end is sufficient. But if 40 people show up, then it becomes tricky.

This initial work lead us the the following results collected in our dashboard presenting:
-    The number of Active Workshops
-    The active workshops by type
-    The total number of registrations from November 1st 2020 till today
-    The total number of Customer (student) registrations
-    The total workshops split
![Hackshack Dashboard](img/hackshack-dashboard.png)

As you can see, moving from a heavily manually oriented approach to now a fully automated one helped increase our numbers.

Of course, as automation minded people, we could not stay like that for long, so we decided to build a framework to help us deliver more Workshops on Demand (aka WoD) and began our 5 years journey of developing it till version 1.0.0 :-)

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
- Server configuration (based on an existing deployed Operating System with just git installed) including:
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
- Server conformity management (run after the previous playbook and on demand and nightly to ensure that the configuration is consistent and up to date):
    - System Update
    - Security Setup
    - Services Conformity and check
    - Templating of configration and data files
- Notebooks deployment on the JupyterHub server:
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
- [wod-doc](https://github.com/Workshops-on-Demand/.github) for the project documentation (you're on it !)

## How it works

Let me first show you how the overall process works for our Workshops-on-Demand:

![Workshops-on-Demand principles](img/wod-principles.png)

If you’re looking for a live explanation of this automation, you can review [the following session](https://www.youtube.com/watch?v=D6Ss3T2p008&t=515s) delivered at Linuxconf in Australia in January 2021. If you prefer reading, consider our [User's Guide](USER-GUIDE.md)

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


