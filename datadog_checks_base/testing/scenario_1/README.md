# Testing release
To check dependencies `datadog_checks_base/datadog_checks/data/agent_requirements.in`, there are two things that can be done
first checkout the `integrations-core` repo and checkout the testing tag.
Then run `ddev dep freeze` - If it says no changes, then it means that the file is in sync with what's in each integrations

The second way to check is to run the agent.
You can use the docker agent

Then ssh into the container and run `/opt/datadog-agent/embedded/bin/pip freeze`
Manually check that the outputed list match the file in the repo