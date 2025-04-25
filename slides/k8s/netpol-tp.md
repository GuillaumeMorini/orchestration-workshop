# TP k8s : Network policies

- We will connect to the Kubernetes master VM

- We will clone the TP repository

.lab[

- Clone the repository on your VM:
  ```bash
  git clone https://github.com/GuillaumeMorini/TP-Kubernetes
  ```

]

- This repository has the applications to deploy but no network policies.

---

## What's this application?


- The demo for this part of the workshop is a distributed application with multiple components. 

- Version 1 of the app has a legacy backend service, which is fronted by a facade. There's a client application which talks to the facade to use backend services.

- Version 2 of the app introduces a payment service. The service isn't directly accessible - only via a queue. Backend services can publish request messages to the queue, and the payments service can process them and publish response messages to the queue.

---

class: pic

![Diagram showing the applications](images/netpol-app.png)

---

class: pic

![Diagram showing the applications v2](images/netpol-app-2.png)

---

## Why using Network policies

--

- The app has been badly configured though, so every service is trying to use every other service. 
- We'll use network policies enforced by Calico to make sure traffic flows where it should.


---

## Goal of the TP

- Deploy the app v1 using kubectl commands

- Write 4 network policies to filter the traffic

- Deploy the app v2 using kubectl commands

- Write 3 network policies to filter the traffic specific to this v2

---

## What I ask

- Work by group on this TP

- Provide a detailed report of all the steps of this TP

- One report per group (be sure to put the 2 or 3 names on the report)

- Use PDF format for the report

- Send me the report through Teams private message or through email

---

## Up to you


![The floor is yours](images/floor_is_yours.jpg)