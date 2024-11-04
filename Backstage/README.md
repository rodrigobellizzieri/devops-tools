# What is Backstage?

Backstage is an open-source developer platform created by Spotify to centralize and simplify the management of tools and infrastructure used by engineering teams. It organizes and makes all services, applications, and components accessible in a single portal, promoting standardization and ease of access. With a user-friendly interface and modular plugins, Backstage allows developers to integrate tools like CI/CD, documentation, monitoring, and more, all in one unified dashboard. It is especially useful for managing microservices and complex infrastructures, enhancing developer productivity and experience.

Official Doc: [Backstage Docs](https://backstage.io/docs/overview/what-is-backstage) 

<br>


## Setup Backstage Development Environment

<br>
<br>

### Install & Configure NVM

NVM - Node Version Manager, nvm allows you to quickly install and use different versions of Node via the command line.

1. Install NVM
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
2. Install Node 
```
nvm install 20
```
3. Use Node version 20
```
nvm use 20
```

<br>
<br>

##

### Install Yarn
1. Install Yarn
```
npm install --global yarn
```
2. Set Yarn to version 4
```
yarn set version 4.4.1
```
3. Verify the Yarn version
```
yarn --version
```

<br>
<br>

##
### Create your Backstage Application
1. Create a Backstage application
```
npx @backstage/create-app@latest
```
2. cd to your application directory (based on the given name)
```
cd backstage/
```
3. Run backstage in a development mode
```
yarn dev
```

<br>
<br>

##
### Verify your application
1. Go to the Backstage UI (should open automatically after the `yarn dev` command)
2. Import a component using this file by going into [Register an existing component](http://localhost:3000/catalog-import) and import this file- `https://github.com/backstage/backstage/blob/master/catalog-info.yaml`
3. Review the example component

<br>
<br>

## Run Backstage On Kubernetes
### Build the container image
1. In your application directory, build your application
```
yarn install --frozen-lockfile
yarn tsc
yarn build:backend
```
3. Build the image
```
docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.0
```
<br>

##
### Upload the image to your registry/kind cluster

This example is for Kind cluster, you can import the image to your docker repository

```
kind load docker-image backstage:1.0.0 --name local-single-node
```

<br>

##
### Deploy Postgres

We're going to use a local Postgres instance, you can use any postgres instance for it.

1. Create the backstage namespace
```
kubectl create ns backstage
```

2. Apply the manifests in the directory from this repo `portgres-resources`. It will create the Postgres resources required with a default username-password.
```
kubectl apply -f postgres-resources
```

Now we need test our Postgres
1. Verify the access to Postgres
```bash
export PG_POD=$(kubectl get pods -n backstage -o=jsonpath='{.items[0].metadata.name}')
```
```bash
kubectl exec -it --namespace=backstage $PG_POD -- /bin/bash

bash-5.1# psql -U $POSTGRES_USER
psql (13.2)
backstage=# \q
bash-5.1# exit
```
##
<br>

### Deploy Backstage on Kubernetes

1. Create the Backstage resources by preparing the files the apply them to the target cluster. You can find the instructions for it in the docs website as well - (link)[https://backstage.io/docs/deployment/k8s#creating-the-backstage-instance]


1. Edit `backstage-resources/bs-secret.yaml` with your github api token. token must have the permissions explained (here)[https://backstage.io/docs/integrations/github/locations/#token-scopes].

1. Apply the Backstage manifests
``` bash
kubectl apply -f backstage-resources
```
##
<br>

## Fix possible error 

If you received an error as shown below, follow the steps to fix it:

``` bash
Error Example:

Error: Failed to connect to the database to make sure that 'backstage_plugin_app' exists, error: password authentication failed for user "backstage"
    at PgConnector.getClient (/app/node_modules/@backstage/backend-defaults/dist/entrypoints/database/connectors/postgres.cjs.js:143:15)
```
1. Access your Postgres pod
``` bash
kubectl exec -it $PG_POD -n backstage -- /bin/bash
```

2. Now login in to your Postgres
``` bash
psql -U $POSTGRES_USER
```

3. After this, you need to change your password. You can set it to the same password that you used in your PostgreSQL secret:
``` bash
ALTER USER backstage WITH PASSWORD 'nova_senha';
```
After this, your Backstage pod should become stable. If necessary, you can delete the Backstage pod so that the ReplicaSet creates a new pod.


1. Check your running instance by port forwarding to it
``` bash
kubectl port-forward --namespace=backstage svc/backstage 8080:80
```

2. access the backstage app: http://127.0.0.1:8080/
##

<br>

### Add login as Guest
1. Everything you need to do to set this up is to add the following code to your app-config.yaml:
``` bash
auth:
  providers:
    guest: 
      dangerouslyAllowOutsideDevelopment: true
```
If possible, please read the documentation: [Auth Doc](https://backstage.io/docs/auth/)

2. After this you can need build and push your image again
``` bash
yarn install --frozen-lockfile
yarn tsc
yarn build:backend

docker image build . -f packages/backend/Dockerfile --tag backstage:1.0.1
```
This example is for Kind cluster, you can import the image to your docker repository

``` bash
kind load docker-image backstage:1.0.1 --name local-single-node
```

3. Now set this image in your backstage-resources/bs-deploy.yaml and make the deploy again
``` bash
kubectl deploy -f backstage-resources
```

## Now you can play with Backstage 🕹


I hope help you, and if you have any question you can see the official doc or send me a message in my Linkedin

[Backstage Doc](https://backstage.io/docs/overview/what-is-backstage)

[Linkedin: Rodrigo Bellizzieri](https://www.linkedin.com/in/rodrigobellizzieri/)


