# Kubernetes test plan for hardning

*instruction*: write 200 words describing a Kubernetes-specific test plan for hardening your cluster.

Answer these two questions in your test plan:
 - How will you test the changes?
 - How will you ensure the changes don't negatively affect your cluster?

## Test plan
My plan is to create a copy cluster #!/bin/bash

# Stop Docker
sudo systemctl --user stop docker

# Define the new location for the Docker data directory
new_data_dir="/run/media/nadavsinai/LinData/docker_images"

# Create the new directory
sudo mkdir -p "$new_data_dir"

# Copy the existing Docker data to the new location
sudo rsync -aP --info=progress2 --exclude='*.log' ~/.local/share/docker/ "$new_data_dir"

# Set the ownership and permissions
sudo chown -R $USER:$USER "$new_data_dir"
sudo chmod -R 700 "$new_data_dir"

# Create or update the Docker configuration file
docker_config="$HOME/.config/docker/daemon.json"
mkdir -p "$(dirname "$docker_config")"
echo '{"data-root": "'"$new_data_dir"'"}' | tee "$docker_config"

# Start Docker
sudo systemctl --user start docker
of my production cluster so I can safely test the changes.
All the actions to the creation of the cluster are going to be done by running declarative configuration files (cluster.yml) and not by running commands on the cluster.
This way I can easily recreate the cluster and test the changes.
My cluster configuration will be stored in a git repository (IaC repo ) so I can easily track the changes and revert them if needed.
Beyond the actions done on every commit which validate the cluster state is as expected (eg via terraform), I will run the following tests:
 - Run kube-bench to validate the cluster is hardened
 - deploy a simple application and run an automated security scan on it to validate the cluster is hardened
 - deploy the actual application and run
    - security scan
    - load/performance test
    - automated UI test

Since the tests will be running on every commit to the repo and I will make sure they pass before doing any changes, the tests will stop me from mergeing unsafe changes which affect my applications security,performance or behaviour.

