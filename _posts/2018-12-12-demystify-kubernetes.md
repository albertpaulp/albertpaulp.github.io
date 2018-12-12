---
layout: slide
title: Demystify Kubernetes
description: Presentation about Why, What and How Kubernetes !
theme: league
transition: fade
category: presentation
---
<div class="reveal">
	<div class="slides">
    <section data-markdown>
      <textarea data-template>
        ## Demystifying Kubernetes
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ## Issues of traditional application management 

        - Unexpected application crash <!-- .element: class="fragment" data-fragment-index="1" -->
        - Resource saturation: usually CPU, Memory and diskspace<!-- .element: class="fragment" data-fragment-index="2" -->
        - Traffic surge ! <!-- .element: class="fragment" data-fragment-index="3" -->
        - Hardware/Cloud provider failures <!-- .element: class="fragment" data-fragment-index="5" -->
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Fixing these in traditional manner

        - Waking up in the middle of the night ! <!-- .element: class="fragment" data-fragment-index="1" -->
        - Ping the server to see if it's up ? <!-- .element: class="fragment" data-fragment-index="2" -->
        - Check the logs or analyze the issue ? <!-- .element: class="fragment" data-fragment-index="3" -->
        - Restart App or Provision more h/w (-+)<!-- .element: class="fragment" data-fragment-index="4" -->
        - Check if application metrics are satisfactionary. <!-- .element: class="fragment" data-fragment-index="5" -->
        - Apart from calling 1800-DEPLOYMENT !
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Aftermath

        - Humans add latency which causes increased MTTR. <!-- .element: class="fragment" data-fragment-index="1" -->
        - Possibility of making mistakes ! <!-- .element: class="fragment" data-fragment-index="2" -->
        - Business taking a hit ! 🔥 <!-- .element: class="fragment" data-fragment-index="3" -->
        - Developer Happiness goes down the hill ! 🤯 <!-- .element: class="fragment" data-fragment-index="4" -->
        - Dev effort grows propotional to no./size of application <!-- .element: class="fragment" data-fragment-index="5" -->
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Building the perfect system

        - Remove "humans" from equation 🤖 <!-- .element: class="fragment" data-fragment-index="1" -->
        - System should be proactive NOT reactive. <!-- .element: class="fragment" data-fragment-index="2" -->
        - System has to understand the rules of application. <!-- .element: class="fragment" data-fragment-index="3" -->
        - Ability to scale hardware and software.  <!-- .element: class="fragment" data-fragment-index="4" -->
        - All of this should be done without waking up developer ! <!-- .element: class="fragment" data-fragment-index="5" -->
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ![morpheus](/assets/images/morpheus.jpg)
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ## Enter Kubernetes
        ![budha](/assets/images/budha.jpg)
        - Image sourced from fineartamerica.com
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Pre-requisites for Kubernetes

        - Kubernetes doesn't know how to run an application, It only knows to spin up container images. <!-- .element: class="fragment" data-fragment-index="1" -->
        - Can containarize applications with Docker and pass it to Kubernetes. <!-- .element: class="fragment" data-fragment-index="2" -->
        - Kubernetes Configuration YAML DSL <!-- .element: class="fragment" data-fragment-index="3" -->
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Container

        - Container are a isolated runtime which runs on host, created by running images.
        - We use Dockerfiles to create images.
        ![pods](/assets/images/container.png)
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Pods

        - Kubernetes abstraction of containers to provide additional configurations to containers.
        ![pods](/assets/images/pods.svg)
      </textarea>
    </section>


    <section data-markdown>
      <textarea data-template>
        ### Hands On setup !

        - brew install docker
        - brew cask install minikube
        - brew cask install virtualbox [Give permission to add virtualization driver]
        - minikube start
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### How Kubernetes works !

        - K8 knows how to deploy containers. <!-- .element: class="fragment" data-fragment-index="1" -->
        - K8 knows how to check health of application. <!-- .element: class="fragment" data-fragment-index="2" -->
        - K8 can restart/reschedule application if case if it's unhealthy. <!-- .element: class="fragment" data-fragment-index="3" -->
        - K8 can scale up/down underlying hardware and software. <!-- .element: class="fragment" data-fragment-index="4" -->
        - K8 can be configured for a High Availability architecture. <!-- .element: class="fragment" data-fragment-index="5" -->
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Kubernetes Architecture

        - One master node to rule 'em all. (Can have multiple master for HA)
        ![architecture](/assets/images/architecture.png)
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Kubernetes Deployment Components

        - Deployment configuration
        - Service configuration
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Demo Time !

        - Use minikube if that's setup.
        - If not, use katacoda.com/courses/kubernetes/playground
        - Create deployment <!-- .element: class="fragment" data-fragment-index="1" -->
        - Show Fault tolerence <!-- .element: class="fragment" data-fragment-index="2" -->
        - Show scaling <!-- .element: class="fragment" data-fragment-index="3" -->
      </textarea>
    </section>

    <section data-markdown>
      <textarea data-template>
        ### Thank You for your time :pray:
        ### Questions, keep 'em coming..


        @albertpaulp
      </textarea>
    </section>
  </div>
</div>
