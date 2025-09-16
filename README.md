# Blue-Green Github Action
Branch "action"


SonarQube Setup on Local
docker run -d --name sonarqube -p 9000:9000 -v sonarqube_data:/opt/sonarqube/data -v sonarqube_extensions:/opt/sonarqube/extensions -v sonarqube_logs:/opt/sonarqube/logs sonarqube

# Runner Setup
	
Download
# Create a folder
$ mkdir actions-runner && cd actions-runner 

# Download the latest runner package
$ curl -o actions-runner-linux-x64-2.328.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.328.0/actions-runner-linux-x64-2.328.0.tar.gz 

# Optional: Validate the hash
$ echo "01066fad3a2893e63e6ca880ae3a1fad5bf9329d60e77ee15f2b97c148c3cd4e  actions-runner-linux-x64-2.328.0.tar.gz" | shasum -a 256 -c

# Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.328.0.tar.gz

Configure
# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/arvindpdige/Blue-Green --token ANPUMXELHRSBJSNPDZJSHO3IZEKJQ

# Last step, run it!
$ ./run.sh

Using your self-hosted runner
# Use this YAML in your workflow file for each job
runs-on: self-hosted

########### Github Runner only runs if we provide the matching labels, it won't take runner name ######

# To get SOnarQubeScan steps
https://github.com/marketplace/actions/sonarqube-quality-gate-check

# To get build & push docker image steps
https://github.com/marketplace/actions/build-and-push-docker-images


