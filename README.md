## Guestbook Example with [Wercker](http://wercker.com) integration


This example shows how to build a simple, multi-tier web application using [Helm](https://helm.sh), [Deis Workflow PaaS](https://deis.com/) and [Wercker](http://wercker.com) for Continuous Deployment

The example consists of:

- A web frontend which is installed as a Deis Workflow PaaS App
- And a back-end [Redis](http://redis.io/) `master` (for storage) with a replicated set of Redis `slaves`.
- Integration with [Wercker](http://wercker.com) for Continuous Deployment of your App's docker image

The web frontend interacts with the Redis `master` API via JavaScript API calls.

-

### Prerequisites

This example requires a running [Kubernetes](https://kubernetes.io) cluster and you have installed [Helm](https://helm.sh), [Deis Workflow Paas](https://github.com/deis/workflow) and you have an account with [Wercker](http://wercker.com).


-
#### Backend install with [Helm](https://helm.sh)

1) We add the remote repo to Helm:
```
$ helm up
$ helm repo add demo-charts https://github.com/deis/demo-charts
$ helm up
```

2) We install our back-end chart
```
$ helm fetch demo-charts/redis-guestbook
$ helm install redis-guestbook
```

#### Front-end install with deis cli

1) Create guestbook App:
```
$ deis create guestbook --no-remote
```

**Note**: We are creating the App with `--no-remote`, as later one we are going to use docker image built by Wercker

2) Set `env vars` so the App knows where to connect to redis cluster:
```
$ deis config:set GET_HOSTS_FROM=env REDIS_MASTER_SERVICE_HOST=redis-master.default REDIS_SLAVE_SERVICE_HOST=redis-slave.default -a guestbook
```
 

#### Set Continuous Deployment with [Wercker](http://wercker.com) 

1) Fork the repo `https://github.com/deis/example-guestbook-wercker.git` to your Gihub account

2) Clone the repo:

```
$ git clone https://github.com/your_github_account/example-guestbook-wercker.git
```

3) Create a new [Wercker App](http://devcenter.wercker.com/docs/web-interface/adding-a-new-application.html) and connect to your App's Github repo

4) Under App's [Settings->Environment variables](http://devcenter.wercker.com/docs/environment-variables/creating-env-vars.html) create the values below:

```
DOCKER_USERNAME: dockerhub or other hosted docker registry account name
DOCKER_PASSWORD: your docker registry account password
DOCKER_REPO: your docker repository e.g. your_docker_hub_user_name/wercker-demo-app
DEIS_CONTROLLER: Deis PaaS controller URL e.g. http://deis.example.com
DEIS_TOKEN: your Deis PaaS user token, which you can get from ~/.deis/client.json file
```
The `Environment variables` above will be used by Wercker App you have created reading pipeline steps from the  [wercker.yml](wercker.yml) in your App's repository


5) Make changes to your code, push to Github and Wercker will do the following:

```
- build the docker image
- tag it
- push to your docker registry repository
- pull the new docker image on your remote Deis Workflow PaaS
```

-

With this example App you have learned how to set the multi-tier web application up using Helm and Deis Workflow and then with Wercker's help to deploy your App to Deis Workflow
