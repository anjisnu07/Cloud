# Getting Started: Create and Manage Cloud Resources: Challenge Lab
# --AIR

## Task 1: Create a project jumphost instance

💙  NAVIGATION BAR => COMPUTE ENGINE => VM INSTANCE => CREATE VM INSTANCE
## Task 2: Create a Kubernetes service cluster

:purple_heart: Run the following from the Cloud Terminal:

:blue_heart: gcloud config set compute/zone us-east1-b
:blue_heart: gcloud container clusters create nucleus-backend \
          --num-nodes 1 \
          --network nucleus-vpc \
          --region us-east1

:blue_heart: gcloud container clusters get-credentials nucleus-backend \
          --region us-east1

:blue_heart: kubectl create deployment hello-server \
          --image=gcr.io/google-samples/hello-app:2.0

:blue_heart: kubectl expose deployment hello-server \
          --type=LoadBalancer \
          --port 8080

## Task 3: Setup an HTTP load balancer

:purple_heart: Run the following from the Cloud Terminal:

:blue_heart: cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

:blue_heart: gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network nucleus-vpc \
          --machine-type g1-small \
          --region us-east1

:blue_heart: gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region us-east1

:blue_heart: gcloud compute firewall-rules create Firewall rule \
          --allow tcp:80 \
          --network nucleus-vpc

:blue_heart: gcloud compute http-health-checks create http-basic-check

:blue_heart: gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region us-east1

:blue_heart: gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global

:blue_heart: gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-region us-east1 \
          --global

:blue_heart: gcloud compute url-maps create web-server-map \
          --default-service web-server-backend

:blue_heart: gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map

:blue_heart: gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80

:blue_heart: gcloud compute forwarding-rules list
