Kubernetes Production-Ready Nginx Deployment
Author: Santhosh reddy kistipati
Project Mission
My goal with this project was to move beyond a basic "Hello World" deployment. I wanted to build a system that reflects how real-world enterprise applications are orchestrated: isolated, secure, self-healing, and resource-aware.

I didn't just want an Nginx page; I wanted an Nginx infrastructure that could survive a crash and protect its credentials.

Technical Architecture & My Decisions
1. Environment Isolation (Namespaces)
I started by creating a dedicated production namespace.

The Logic: In a real company, you don't want your testing or development pods mixing with your live users. By using a namespace, I’ve created a logical "fence" around this application to keep it organized and secure.

2. High Availability & Self-Healing (Probes)
This is where I spent the most time. I configured two types of health checks:

Liveness Probe: I noticed that sometimes Nginx can be "running" but not actually serving traffic. I set up a "doctor" check that pings the port every 10 seconds. If it fails 3 times, Kubernetes kills the pod and starts a fresh one. No manual intervention needed.

Readiness Probe: I added this to solve the "half-started" problem. It ensures that the LoadBalancer doesn't send a single user to a pod until the Nginx server is 100% ready to respond.

3. Security vs. Configuration (Secrets & ConfigMaps)
I made a clear distinction between Public and Hush-Hush data:

ConfigMap (cm.yaml): I used this for the index.html. This allows me to change the website content without ever touching the Nginx code or rebuilding the container image.

Secret (secret.yaml): For the database credentials, I used a Secret. Even though they look encoded (Base64), the real benefit I implemented is that they stay in the node's memory and aren't written to the disk, keeping them away from prying eyes.

4. Resource Governance (Limits & Requests)
To prevent my cluster from crashing due to a "greedy" pod, I set hard boundaries:

Requests: I guaranteed 64MB of RAM so the app always has room to breathe.

Limits: I capped it at 128MB. If the app has a memory leak, Kubernetes will shut it down before it slows down my entire laptop/server.

The "Lab" Setup (How I Ran This)
I developed and tested this using Minikube and kubectl.

Deployment Order:
kubectl apply -f namespace.yaml (Built the garage)

kubectl apply -f cm.yaml (Added the content)

kubectl apply -f secret.yaml (Added the keys)

kubectl apply -f deployment.yaml (Started the engines)

kubectl apply -f service.yaml (Opened the front door)

Lessons Learned
While building this, I ran into several YAML indentation issues—specifically with the probes. I learned that Kubernetes is extremely strict about how these sections are nested. Fixing those errors helped me understand the spec.template.spec hierarchy much more deeply.
