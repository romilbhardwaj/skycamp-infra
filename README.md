# SkyCamp Infrastructure Provisioning
Tool to provision infrastructure for SkyCamp 2022 tutorials using SkyPilot.

* Assumes Docker image for you tutorial is built and pushed to some container registry
* ~~Generates a connect script that you can share with attendees. This connect script contains the SSH key and the port forwarding~~. Update 10/19/22 - It still generates connect.sh, but it is recommended to just open the requisite ports on the VM's security group so attendees can easily connect. 

## SkyCamp tutorial lead responsibilities
1. Create a docker image and push it to a public container registry.
2. At runtime, your container will be mounted with a volume at `/credentials`. This volume contains the `.aws` and `.config/gcloud` directories for accessing shared credentials. 
3. One port on this container will be directly port-forwarded to the attendee's laptop. Choose whichever service you want to expose over this. It can be anything, for example `jupyter lab` or `ssh` (which you can further use to expose more ports if necessary).
4. See SkyPilot tutorial for [example](https://github.com/skypilot-org/skypilot-tutorial/blob/skycamp22/Dockerfile)

## SkyCamp Infra manager's responsibilities
Infra manager is one chosen person with access to the SkyCamp AWS and Google Credentials.

1. Install SkyPilot `pip install -r requirements.txt`
2. Setup the shared AWS and GCP credentials locally. See [SkyPilot docs](https://skypilot.readthedocs.io/en/latest/getting-started/installation.html).
3. Add everyone's docker image to the `run` section in provision_tutorial_docker.yaml
4. Run `sky launch -c tutorial provision_tutorial_docker.yaml`. Wait for provisioning to finish.
5. After successful provisioning, manually open AWS web console and open ports 8888, 8889 and 8890 on the VM security group (all vms share the sg).
6. Run `python get_ssh_creds.py tutorial`. This script will generate a CSV with the list of IP addresses for the provisioned machines and a `connect.sh` script which contains the shared SSH key.
7. This `connect.sh` is shared amongst all attendees, so you can email all attendees with this script.
8. `ssh_creds.csv` contains IP addresses of the VMs you created. 
9. Use google apps script [mail merge](https://developers.google.com/apps-script/samples/automations/mail-merge) to send out bulk emails with IPs. See this template:
```
Hi {{First name}},

Welcome to SkyCamp 2022! Your personal VM for running the tutorials is: {{ip}}.

Each tutorial is hosted at a different URL. To access the tutorial notebooks, please open the browser and open:

Skyplane tutorial: http://{{ip}}:8888
SkyPilot tutorial: http://{{ip}}:8889
MC2 tutorial: http://{{ip}}:8890

If prompted, the access token for the notebooks is SkyCamp2022.

Note: You may not be able to access these URLs on the public CalVisitor wifi. Please use the "eduroam" wifi network. If you need the wifi credentials, please reach out to a tutorial assistant or the registration desk.

Fun fact: all tutorial VMs for this event were provisioned in one click by SkyPilot!

Best,
SkyPilot Team
```
9. After the tutorial, run `sky down tutorial` to terminate all the machines.

NOTE - Make sure attendees connect to `eduroam` (Ask Jon to generate temp account). `CalVisitor` wifi does not allow any port other than 80 and 443.

## Attendee UX
See email above.

~~1. Download the `connect.sh` script from the email.~~
~~2. Run `./connect.sh <IP_Address>` to connect to the VM. Keep this terminal open.~~
~~3. Now open `http://localhost:8888` in your local browser to access jupyter lab for tutorial 1. To start tutorial 2, increment the port number to `http://localhost:8889`.~~
