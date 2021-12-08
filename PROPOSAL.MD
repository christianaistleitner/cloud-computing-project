# Proposal: Implementation of our own Continuous Deployment (CD) Pipeline

Authors: Christian Aistleitner, Marcel Brunnbauer, Martin Zwifl

Last updated: 08.12.2021 12:57

## Abstract

This project is an assignment for the course 510.211 Special Topics - Cloud Computing (2021W), held at the Johannes Kepler University, Linz.
It aims for gaining additional hands-on experience and therefore, we concentrate on implementing our own CD pipeline. 


## Scope
Our goal is to implement a CD pipeline for a simple demo web app.
Therefore, we are going to use the tool [Tekton](https://tekton.dev/) for implementing the CD pipeline, being able to deploy artifacts in a dev and production branch.
Implement a pipeline which deploys an artifact in at least a two-stage environment (i.e. dev and production)

## Milestones

- Proposal pulication (Deadline: 08.12.2021)
- Proposal refinement and acception (Deadline: 15.12.2021)
- Cluster and Tekton setup done
- Simple demo app + unit test implemented
- Delivery pipeline working
- Documentation finalized
- Final Presentation on 19.1.2022


## Responsibilities

### [Christian Aistleitner](https://github.com/christianaistleitner)
- Cluster/Tekton setup
- Tekton Dashboard

### [Marcel Brunnbauer](https://github.com/Marcel256)
- Trigger Bindings
- Demo app + unit test

### [Martin Zwifl](https://github.com/martin-zwifl)
- Task config
- Delivery pipeline
