![Firefly logo](https://github.com/try-firefly/.github/blob/main/profile/assets/firefly_logo.png)

##

Firefly is an open-source observability solution for serverless functions. Through automating instrumentation and the deployment of a telemetry pipeline, users can have key insights readily available in an easy-to-use dashboard in a matter of minutes.

## Getting started
[**Setup the Server**](https://github.com/try-firefly/.github/edit/main/profile/README.md#setup-the-server)

- You'll have to setup a server to host our metrics/traces endpoint. You'll need a single URL to point data to when you run the command line tool later on.

[**Setup Pipeline Infrastructure**](https://github.com/try-firefly/.github/edit/main/profile/README.md#setup-pipeline-infrastructure)

- Clone the `firefly-pipeline` onto the same machine that you host the server on and spin up the containers.



*Once you have the server setup you can move on to using the firefly command line tool.*



[**Requirements For Firefly CLI**](https://github.com/try-firefly/.github/edit/main/profile/README.md#requirements-for-firefly-cli)

- you'll need to download terraform and have your aws credentials in a shared file

[**Firefly CLI**](https://github.com/try-firefly/.github/edit/main/profile/README.md#firefly-cli)

- the last step in our setup, clone our repository and run the command line tool. There will be several prompts to fill out details.





## Setup the Server

Once you have spun up the infrastructure there will be a number of ports exposed that require routing. These ports are as follows:

* 3000 -- grafana
* 4433 -- metrics endpoint
* 4318 -- traces endpoint

If you are running a reverse proxy then please find the relevant instructions [here](https://grafana.com/tutorials/run-grafana-behind-a-proxy/) to setup grafana.
As an alternative you can also setup a bespoke subdomain for grafana, this way you only have to handle traffic to one path. For example if you were using `nginx`
your server block could simply be:-

```
location / {
  proxy_pass http://localhost:3000;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
```

Trace data is emitted to your chosen endpoint with the following path appended `/v1/traces`; you will need to route this traffic to port `4318` with the same path. Metric data will be emitted to the same endpoint with no path appended, this traffic can be routed to port `4433` with no path. If you chose to use a bespoke subdomain for your grafana setup, an example `nginx` server block setup for ports `4433` and `4318` can be seen below.

```
location / {
  proxy_pass http://localhost:4433;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}

location /v1/traces {
  proxy_pass http://localhost:4318/v1/traces;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
```

## Setup Pipeline Infrastructure

1. `git clone` [this](https://github.com/try-firefly/firefly-pipeline) repo onto the server with your designated HTTPS address for monitoring
2. `cd` into the directory
3. Run the following command:

```
docker-compose up -d
```

You should now have several docker containers running on your machine.





## Requirements For Firefly CLI

1. AWS account
2. AWS credentials in a [shared file](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/loading-node-credentials-shared.html)
3. [terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)
4. Server with an HTTPS address in which telemetry data can be sent to (you set this up in the above)



## Firefly CLI

1. `git clone` [this](https://github.com/try-firefly/firefly-cli) repo
2. Run `npm install` in the root directory
3. `cd` into the `bin` directory and run `node cli.js`

The `cli` instruments functions based on  region. If you have functions residing in different regions, simply run  the CLI again to setup the necessary infrastructure in that region.
