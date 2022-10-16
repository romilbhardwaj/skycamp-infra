# SkyCamp Infrastructure Provisioning
Tool to provision infrastructure for SkyCamp 2022 tutorials using SkyPilot.

* Assumes Docker image for you tutorial is built and pushed to some container registry
* Generates a connect script that you can share with attendees. This connect script contains the SSH key and the port forwarding

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
5. After successful provisioning, run `python get_ssh_creds.py tutorial`. This script will generate a CSV with the list of IP addresses for the provisioned machines and a `connect.sh` script which contains the shared SSH key.
6. This `connect.sh` is shared amongst all attendees, so you can email all attendees with this script.
7. `ssh_creds.csv` contains IP addresses of the VMs you created. TBD: How to distribute this? Email script? Or hand over the morning of the tutorial during registration? 
8. After the tutorial, run `sky down tutorial` to terminate all the machines.

## Attendee UX
1. Download the `connect.sh` script from the email.
2. Run `./connect.sh <IP_Address>` to connect to the VM. Keep this terminal open.
3. Now open `http://localhost:8888` in your local browser to access jupyter lab for tutorial 1. To start tutorial 2, increment the port number to `http://localhost:8889`.
4. Exit once you're done. 
