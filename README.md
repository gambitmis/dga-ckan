# dga-ckan
# k8s-dga-ckan
Transform Docker Compose of Nectec CKAN to K8s deployment 

```
export project="project-xxxxx"
docker image build -t ckan-solr thai-gdc/solr/
docker image build -t ckan-postgres thai-gdc/postgresql/
docker image tag ckan-postgres gcr.io/$project/ckan-postgres
docker push gcr.io/$project/ckan-postgres
docker image tag ckan-solr gcr.io/$project/ckan-solr
docker push gcr.io/$project/ckan-solr
docker pull thepaeth/ckan-thai_gdc:ckan-2.9-xloader
docker image tag thepaeth/ckan-thai_gdc:ckan-2.9-xloader gcr.io/$project/ckan-gdc
docker image push gcr.io/$project/ckan-gdc
docker image tag redis gcr.io/$project/ckan-redis
gcloud compute disks create --size=10G --zone=us-central1-c domain-x-postgres
gcloud compute disks create --size=10G --zone=us-central1-c domain-x-ckan
gcloud compute disks create --size=10G --zone=us-central1-c domain-x-solr
```

Prepare
```
export project="open-data-359610"
```

Build Solr Image and Push to Registry
```
docker image build -t ckan-solr thai-gdc/solr/
docker image tag ckan-solr gcr.io/$project/ckan-solr
docker push gcr.io/$project/ckan-solr
```

Build Postgres
```
docker image build -t ckan-postgres thai-gdc/postgresql/
docker image tag ckan-postgres gcr.io/$project/ckan-postgres
docker push gcr.io/$project/ckan-postgres
```

Build ckan
```
docker pull thepaeth/ckan-thai_gdc:ckan-2.9-xloader
docker image tag thepaeth/ckan-thai_gdc:ckan-2.9-xloader gcr.io/$project/ckan-gdc
docker image push gcr.io/$project/ckan-gdc
```

Build Redis
```
docker image pull redis
docker image tag redis gcr.io/$project/ckan-redis
docker image push gcr.io/$project/ckan-redis
```

```
gcloud compute disks create --size=10G --zone=asia-southeast1-a	 test-1-postgres
gcloud compute disks create --size=10G --zone=asia-southeast1-a	 test-1-solr
```

```
kubectl -n dga-ckan exec -it deploy-postgres-68b79b4b58-r2p9z -c postgres -- sh
pg_dump -O -x --format=c -d ckan > db_ckan.dump -U ckan
pg_dump -O -x --format=c -d datastore > db_data.dump -U ckan

kubectl -n dga-ckan exec -it ckan-1-k4txd -c testimage -- sh
tar -czvf /tmp/ckan.tar.gz /var/lib/ckan

kubectl -n dga-ckan cp ckan-1-k4txd:/tmp/ckan.tar.gz /tmp/ckan-web.tar.gz -c testimage

kubectl -n dga-ckan cp deploy-postgres-68b79b4b58-r2p9z:/tmp/db_ckan.dump /tmp/ckan.dump -c postgres
kubectl -n dga-ckan cp deploy-postgres-68b79b4b58-r2p9z:/tmp/db_data.dump /tmp/data.dump -c postgres
```

```
https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html
https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html
https://docs.aws.amazon.com/eks/latest/userguide/add-ons-images.html

```

```
kubectl -n yalacity exec deploy-postgres-xxxx -- /usr/bin/pg_dump -O -x --format=c -d ckan --file=/tmp/yalacity-ckan.dump -U ckan
kubectl -n yalacity exec deploy-postgres-xxxx -- /usr/bin/pg_dump -O -x --format=c -d ckan --file=/tmp/yalacity-data.dump -U ckan
```

```
#!/bin/bash
  
ns="yalacity"
datetime=$(date +"%Y%m%d-%H")

ckan=$(kubectl -n "$ns" get pod -o=jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | grep ckan)
pg=$(kubectl -n "$ns" get pod -o=jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}' | grep postgres)

echo "Ckan container name $ckan"
echo "Postgres container name $pg"
echo "Namespace $ns"
echo "###################################"

if kubectl -n "$ns" exec "$pg" -- /usr/bin/pg_dump -O -x --format=c -d ckan --file="/tmp/$ns-ckan-$datetime.dump" -U ckan; then
    echo "Dump file CKAN created successfully."
else
    echo "Error creating dump file."
fi

if kubectl -n "$ns" exec "$pg" -- /usr/bin/pg_dump -O -x --format=c -d datastore --file="/tmp/$ns-datastore-$datetime.dump" -U ckan; then
    echo "Dump file DATASTORE created successfully."
else
    echo "Error creating dump file."
fi

if kubectl -n "$ns" cp "$pg":/tmp/$ns-ckan-$datetime.dump /tmp/$ns-ckan-$datetime.dump; then
    echo "Download CKAN dump to /tmp already."
else
    echo "Error can not download file."
fi

if kubectl -n "$ns" cp "$pg":/tmp/$ns-datastore-$datetime.dump /tmp/$ns-datastore-$datetime.dump; then
    echo "Download DATASTORE dump to /tmp already."
else
    echo "Error can not download file."
fi

if kubectl -n "$ns" exec "$ckan" -c testimage -- tar -czvf /tmp/$ns-webckan-$datetime.tar.gz /var/lib/ckan; then
    echo "Backup file Web-CKAN created successfully."
else
    echo "Error creating dump file."
fi

if kubectl -n "$ns" cp "$ckan":/tmp/$ns-webckan-$datetime.tar.gz /tmp/$ns-webckan-$datetime.tar.gz -c testimage; then
    echo "Download backup Web-CKAN created successfully."
else
    echo "Error download backup file."
fi
```
