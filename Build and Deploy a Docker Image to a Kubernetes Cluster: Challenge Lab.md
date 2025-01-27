#Task 1. Create a Kubernetes cluster
gcloud container clusters create echo-cluster --num-nodes 2 --zone us-east1-d --machine-type e2-standard-2


#Task 2. Build a tagged Docker image
gsutil cp gs://$DEVSHELL_PROJECT_ID/echo-web.tar.gz .

tar -xvf echo-web.tar.gz

gcloud builds submit --tag gcr.io/$DEVSHELL_PROJECT_ID/echo-app:v1 .


#Task 3. Push the image to the Google Container Registry
docker push gcr.io/${PROJECT_ID}/echo-app:v1



#Task 4. Deploy the application to the Kubernetes cluster
kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1

kubectl expose deployment echo-web --type=LoadBalancer --port=80 --target-port=8000



