---
layout: post
title: Use Clojure with Docker and Kubernetes!
category: [clojure]
tags: [devops, automation, ci, container, docker, kubernetes, cloud]
date: 2020-08-29
---


![Cursive REPL output window](/img/2020-08-29-use-clojure-with-docker-and-kubernetes_img_1.png)

*Cursive REPL output window. Just listing the buckets using my repl scratch file.*

### Introduction

At my new company [Metosin](http://www.metosin.fi) I have been implementing [AWS](https://aws.amazon.com/) infrastructure using [Pulumi](https://www.pulumi.com/) and implementing applications using [Clojure](https://clojure.org/). We are deploying applications into a [Kubernetes](https://kubernetes.io/) cluster and to verify that our applications run with the infrastructure in a dockerized environment as they should we are running tests locally using Drone pipeline (see my previous blog post about it: [Using Drone CI]({% post_url 2020-08-26-using-drone-ci %}) ) which spins the required infrastructure using docker containers ([Postgres](https://www.postgresql.org/) and [Minio](https://hub.docker.com/r/minio/minio/)) and the actual application as another [docker container](https://www.docker.com/resources/what-container). This week I was wondering that one application behaved a bit differently running tests in my local workstation hitting containers using localhost and running tests inside a docker container. I was discussing this with one Metosin colleague and he told me how to add an [nrepl server](https://github.com/nrepl/nrepl) inside the application so that I can connect to the application running in a docker container and experiment with the running application there using my [IntelliJ IDEA](https://www.jetbrains.com/idea/) / [Cursive REPL](https://cursive-ide.com/userguide/repl.html). Because this trick was pretty neat I thought that I write a short blog post about it. (By the way, I haven't regretted a second joining Metosin - one of the best places to be if you want to learn Clojure with the best clojurians on the planet, see more: [My First Weeks at Metosin!]({% post_url 2020-03-23-my-first-weeks-at-metosin %}))


### Setting Up Nrepl Server Inside a Clojure Application

Our Clojure applications use [Integrant](https://github.com/weavejester/integrant) as a state management library and we have our configuration in an [Aero](https://github.com/juxt/aero) configuration file. Setting up an nrepl server inside the application is pretty straightforward using these libraries. First the configuration file:

```clojure
...
 :backend/nrepl {:bind #profile {:prod nil
                                 :local "localhost"
                                 :docker "0.0.0.0"
                                 }
                 :port 3177}
...
```

So, in prod don't use nrepl, in `local` profile when running application locally use `localhost` and when running application inside a docker container use `0.0.0.0` address. Nrepl server listens to `3177` port.

Then in your Integrant configuration provide functions so that Integrant knows what to do when you want to init, halt, suspend and resume your state:

```clojure
...
(defmethod ig/init-key :backend/nrepl [_ {:keys [bind port]}]
  (if (and bind port)
    (nrepl/start-server :bind bind :port port)
    nil))

(defmethod ig/halt-key! :backend/nrepl [_ this]
  (if this
    (nrepl/stop-server this)))

(defmethod ig/suspend-key! :backend/nrepl [_ this]
  this)

(defmethod ig/resume-key :backend/nrepl [_ _ _ old-impl]
  old-impl)
...
```

So, in `init` we start the nrepl server and in `halt`, we stop it. In `suspend` we store the nrepl instance and in `resume` we use the old nrepl instance - i.e. when you keep hitting that `Integrant reset` hot key (mine is Alt-J) to reset your state you don't want to stop the nrepl server.

### Docker Configuration

You can use e.g. [Docker Compose](https://docs.docker.com/compose/) when providing the docker configuration for your application. Example follows:

```yml
version: '3.7'
services:
  arkisto_image:
    image: local-arkisto:latest
    ports:
      - "3077:3077"
      - "3177:3177"
    networks:
      [clojure-network]
    container_name: arkisto_image
    environment:
      PROFILE: docker
      DB_USERNAME: arkisto
      DB_PASSWORD: arkisto
      DB_HOST: clojuredb
      DB_PORT: 5432
      MINIO_HOST: clojures3
      AWS_ACCESS_KEY_ID: minioadmin
      AWS_SECRET_ACCESS_KEY: minioadmin

networks:
  clojure-network:
    name: clojure-network
```

The important trick here is to port forward the nrepl port `3177` from docker container to your host (your workstation) so that Cursive REPL can connect to it. (NOTE: This is just a test image configuration, I use another Dockerfile for building the real application image running in Kubernetes, of course - therefore using dummy passwords.)

If you wonder why we have the MINIO_HOST in this yaml file, let's see the Aero configuration once more:

```clojure
              ; Endpoint only used in dev - default value that of Minio.
              :endpoint #profile {:prod nil
                                  :dev {:protocol :http
                                        :hostname #or [#env "MINIO_HOST" "localhost"]
                                        :port 9000}}
```

... so, in production the endpoint configuration is nil (not needed, the Kube EKS uses AWS IAM to get access to the S3 buckets...), but in other environments, we simulate S3 using Minio and thus providing the endpoint to Minio server.


### Connecting to the Nrepl Server From Your IDE

I use excellent [Cursive REPL](https://cursive-ide.com/userguide/repl.html) Clojure IDE as an example, but you can connect to the nrepl server pretty much same way using any Clojure IDE.

![Connecting to Nrepl server](/img/2020-08-29-use-clojure-with-docker-and-kubernetes_img_2.png)

*Cursive REPL configuration.*

Then just click the IntelliJ IDEA "Start configuration" button and Cursive REPL connects immediately into your application's nrepl port. Then you can do in the application code whatever you do also locally: reset state, compile namespaces, etc. The picture at the beginning of the blog post shows one REPL output from my repl scratch file while checking if I can connect to the Minio server running in another container in the same docker network. 

Let's try another trick here. Add something like this in your project's `deps.edn` or in `~/.clojure/deps.edn`:

```clojure
 :aliases {
           :kari {:extra-paths ["scratch"]
                  :extra-deps {hashp {:mvn/version "0.1.1"}
... 
```

This is the excellent [Hashp](https://github.com/weavejester/hashp) debugging print tool.

And then in your clojure scratch file send the following form to repl:

```clojure
  (require '[hashp.core])
```

Then you can add `#p` in front of any form you want to have debugging information. Let's add some printouts and see if we are using either AWS Profile or access key / secret key in that docker environment:

```clojure
(defn get-aws-credentials
  "Get aws credentials. If AWS_PROFILE provided, use it. If AWS basic credentials, use them.
  These are only for testing purposes. In real EKS environment do not provide any credentials
  (returns nil) which means that pod uses role based authorization provided in the Pulumi
  configuration."
  [aws-profile aws-basic-credentials]
  (let [{access-key-id :access-key-id
         secret-access-key :secret-access-key} aws-basic-credentials]
    (cond
      aws-profile #p (credentials/profile-credentials-provider aws-profile)
      (and access-key-id secret-access-key) #p (credentials/basic-credentials-provider aws-basic-credentials))))
```

Then send the namespace to repl and reset Integrant state (in my Cursive IDE I have the following hot keys: Alt-N: switch to namespace your cursor is, Alt-M: send namespace to repl, Alt-J: reset Integrant state - takes less than a second to click these three buttons). I see this in my Cursive REPL output windows:

```clojure
Loading src/clj/arkisto/backend/main.clj... done
(integrant.repl/reset)
:reloading ()
#p[arkisto.backend.main/get-aws-credentials:39] (credentials/basic-credentials-provider aws-basic-credentials) => #<cognitect.aws.credentials$basic_credentials_provider$reify__23486@3c5cb0f5>
=> :resumed
```

... so, we are using access key / secret key (as you could see in the docker compose configuration file earlier).


### How About If I Want to Connect to a Clojure Application inside Kubernetes Pod?

For connecting to a nrepl server running in an application running inside Kubernetes cluster I created a simple [Justfile](https://github.com/casey/just) recipe so that my team members can easily connect to the applications:

```bash
# forward nrepl port from kube pod
@kube-nrepl +args='':
   ./infra/scripts/run-nrepl-port-forward.sh {{args}}
```

Just recipe delegates task to the `run-nrepl-port-forward.sh` which does something like this:

```bash
...
MYNS="XXX-${MYAPP}-master"   # provide namespace
MYPODNAME=$(kubectl get pods -n $MYNS -o=jsonpath='{.items[0].metadata.name}')  # parse pod name
echo "Creating port forward from $MYPODNAME nrepl port $REMOTE_PORT to local port $MYPORT..."
kubectl -n $MYNS port-forward $MYPODNAME $MYPORT:$REMOTE_PORT  # forward port
```

... and when I run the Just recipe:

```bash
λ> just kube-nrepl arkisto 6677
Getting pod for arkisto...
Creating port forward from arkisto-master-897d9bd60c-6tm46 nrepl port 3177 to local port 6677...
Forwarding from 127.0.0.1:6677 -> 3177
Forwarding from [::1]:6677 -> 3177
```

... and then provide the same configuration in your Clojure IDE as provided earlier, but this time the local port is `6677`. You can run both repls at the same time and use repl to examine the differences, e.g.

```clojure
; Sending form in repl connected to Docker...
(user/system)
=>
#:backend{:s3 #object[cognitect.aws.client.Client 0x432cd41f "cognitect.aws.client.Client@432cd41f"],
          :bucket {:use-s3 nil,
                   :bucket "arkisto-mainsystem-bucket",
...
; Sending form in repl connected to Kubernetes...
(user/system)
=>
#:backend{:s3 #object[cognitect.aws.client.Client 0x37f83fa "cognitect.aws.client.Client@37f83fa"],
          :bucket {:use-s3 true,
                   :bucket "my-very-secret-arkisto-bucket-xxxx",   ; NOTE: bucket name changed in this blog!
...
```

So, in my docker test environment I'm not using real [AWS S3](https://aws.amazon.com/s3/) but [Minio](https://hub.docker.com/r/minio/minio/) which provides exactly the same api interface so that the clojure code actually doesn't know if we are using real AWS S3 service or a simulated environment (Minio). In test environment I use some bucket name for development purposes (creates a bucket inside Minio), in real life the application gets the bucket name from [AWS Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html). The `use-s3` is just for the Integrant state to create the bucket in Minio if we are running in a test environment (in real AWS environment the bucket is already there, of course):

```clojure
...
  ; NOTE: Pulumi manages infra creation (S3 bucket).
  (when-not use-s3
    (aws/invoke s3-client {:op :CreateBucket :request {:Bucket bucket}}))
  {:use-s3 use-s3 :bucket bucket :s3-client s3-client :clean clean})
...
```

### Conclusions

If you want to learn an excellent functional language which is ready for the cloud era look no further: [Clojure](https://clojure.org/).



*The writer is working at [Metosin](https://www.metosin.fi/) using Clojure in cloud projects. If you are interested to start a Clojure project in Finland or you are interested to get Clojure training in Finland you can contact me by sending email to my Metosin email address or contact me via LinkedIn.*

Kari Marttila

* Kari Marttila's Home Page in LinkedIn: <https://www.linkedin.com/in/karimarttila/>

