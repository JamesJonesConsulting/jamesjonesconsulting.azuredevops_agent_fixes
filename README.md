# Ansible Collection

## `jamesjonesconsulting.azuredevops_agent_fixes`

This collection is for setting up Azure DevOps self-hosted agents on Podman. If you've worked with Podman, there's several 'gotchas'
and this role calls `jamesjonesconsulting.podman_socket_group_permissions` collection which takes care of the one's I've seen and will extend it as others come to light.  This is a useful way to run hybrid jobs on your 
private network and doesn't incur costs associated with using 'shared' agents hosted by Microsoft for a fee. This allows long (you can run 'unlimited' job times if something takes hours to run) and specialized jobs to be 
offloaded to private self-hosted 'agents' and the lightweight jobs in shared agents to mitigate bottlenecks associated with a
single 'shared' agent approach (more costly, slower and may require 'poking holes' into your secure private network).

There are a couple of nuances with this agent...

1. The agent is an unpackaged binary and not a 'native' package. The setup process queries GitHub's API to determine the latest version and 
downloads that binary for installation purposes.
2. Microsoft has not updated their agents to run on .NET 6 yet so 'newer' linux systems need to install the openssl 1.1 compatibility package to support it and requires some 'handling' to get it working on a recent version of linux.

This role only supports one type of agent but will be extended to support environment agent and agents running in your kubernetes cluster
at some point.

Currently, this collection contains two role called 

* `jamesjonesconsulting.azuredevops_agent_fixes.agent_setup` - sets up a podman user group that has permissions to the socket and integrates pretty seemlessly like docker would. It sets up 1 to many installs of the agent on a single host. 
* `jamesjonesconsulting.azuredevops_agent_fixes.agent_remove` - removes the agent

## Usage

### `jamesjonesconsulting.azuredevops_agent_fixes.agent_setup` Role

The role is called like this in an ansible playbook (note: you will need to be running `ansible-core`)

```
include_role:
  name: jamesjonesconsulting.azuredevops_agent_fixes.agent_setup
vars:
  vsts_agents: 
    - pool: PrivateHosted
      name: agentvm_1
      folder: PrivateHosted/agentvm_1
      username: agentuser
  vsts_agent_podman_group: docker
  vsts_agent_url:https://dev.azure.com/myorganization
  vsts_agent_registration_token: abcdefghijklmnopqrstuvwxyz
```

### Options for the ansible collection `agent_setup` role.

* `vsts_agents` - This is an array of settings to pass the name of the `pool`, the `name` of the agent in the pool, the `folder` where it will be installed off `/opt/vsts-agents` on the host and finally the `username` the agent will be running as. Note: an agent will be installed for each element of this array.
* `vsts_agent_podman_group` - This is passed to the `jamesjonesconsulting.podman_socket_group_permissions` collection which does all the 'podman' magic. It's a 'default' so you can skip it if you don't need to override it.
* `vsts_agent_url` - Pass in the url of your organization where the agents will be setup in the pool(s) created.
* `vsts_agent_registration_token` - This the PAT you must create to install Azure DevOps agents.

### `jamesjonesconsulting.azuredevops_agent_fixes.agent_remove` Role

The role is called like this in an ansible playbook (note: you will need to be running `ansible-core`). Note: This role is called in the agent setup
to remove and setup replacement agents.

```
include_role:
  name: jamesjonesconsulting.azuredevops_agent_fixes.agent_remove
vars:
  vsts_username: agentuser
  vsts_systemd_service_name: vsts.agent.myorganization.PrivateHostede.agentvm_1.service
  vsts_agent_working_dir: /opt/vsts-agents/PrivateHosted/agentvm_1
  vsts_agent_registration_token: abcdefghijklmnopqrstuvwxyz

```

### Options for the ansible collection `agent_remove` role.

* `vsts_username` - The `username` the agent will be running as.
* `vsts_systemd_service_name` - The name of the 'systemd' service that was created for this agent. This is needed to clean up the overlay and is required.
* `vsts_agent_working_dir` - The full path to where the agent is located.
* `vsts_agent_registration_token` - This the PAT you must create to install Azure DevOps agents.

## Setup

Usually you have a `requirements.yml` file you can specify the collection like this:

```
collections:
  - name: https://github.com/JamesJonesConsulting/jamesjonesconsulting.jamesjonesconsulting.azuredevops_agent_fixes.git
    type: git
    version: main
```

Or, you can just install it via command line as well like this:

```
ansible-galaxy collection install git+https://github.com/JamesJonesConsulting/jamesjonesconsulting.jamesjonesconsulting.azuredevops_agent_fixes.git,main
```
