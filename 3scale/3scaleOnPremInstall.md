3scale deployments can be of 2 types broadly speaking:
Option 1: Gateway (installed on baremetal, Docker or Openshift) that talks to the SaaS API manager. Every participant should have an account on the 3scale.net domain
Option 2: Fully On-Prem. In this, all components of 3scale are deployed on Openshift: Gateway and API Manager (which includes the Developer Portal)

<b>Note</b>: The instructions below are for Option 2. above, namely: deploy 3scale fully on-prem (AMP)
The official install guide is here: https://support.3scale.net/guides/infrastructure/onpremises20-installation

The below is a cheat sheet.

Step 1: Check that your cluster is up and running and that you can login to the Openshift console

Step 2: Verify that you have free PVs that dont have PV claims on them.
If you DONT HAVE free PVs, do the following steps..
........KAVITHA PUT STEPS HERE TO CREATE PVs and VERIFY......................

Step 3: Create a new project either via the openshift console or using the oc cli

oc login (developer account will suffice, dont need admin)
oc new-project <project_name> say 3scaleamp

Step 4: Create a new app with the AMP template
Note that in the install guide, you access the template from a 3scale rpm but for this lab, we will make it available here.
The AMP template installs:
1. Two built-in API gateways
2. One AMP admin portal and developer portal with persistent storage

oc new-app --file /path/to/amp.yml --param WILDCARD_DOMAIN=<WILDCARD_DOMAIN> --param ADMIN_PASSWORD=3scaleUser
<P><B>Note</B> The wildcard domain should be of the format <yourname or id>.apps.opsday.ocpcloud.com

That is it. Now, we wait for a few minutes for all the 3scale pods to come up.

<p>Step 5: Verification
<p>To access the 3scale API Manager UI, go to:
https://3scale-admin.kavitha.apps.opsday.ocpcloud.com   (use your id)
Login: admin/3scaleUser
Now, you are all set to proceed to 3scale specific labs !

