# Deploy Express application with Mongo Express app and Mongodb database 

1. Start Your kubernetes cluster
2. Deploy your secrets `mongo-secret.yaml`
3. Deploy your headless service for stateful set `headless-service.yaml`
4. Deploy your stateful mongodb set with 3 replicas `mongodb-deploy.yaml`. Ensure all 3 pods are running
5. ssh or exec into your mongo-0 pod and initiate the replica set follow instructions below

   kubectl exec -it mongo-0 -- mongosh \
   rs.initiate() \
   var cfg = rs.conf() \
   cfg.members[0].host="mongo-0.mongo:27017" \
   rs.reconfig(cfg) \
   rs.status() \
   rs.add("mongo-1.mongo:27017") \
   rs.add("mongo-2.mongo:27017")

   **Make sure you can see all replicasets sync and healthy**
   
7. Make the following changes to your web-app.yaml Under container env change ME_CONFIG_MONGODB_SERVER to ME_CONFIG_MONGODB_URL

ie **Make sure to Change** 
 - name: ME_CONFIG_MONGODB_SERVER \
          valueFrom: \
            configMapKeyRef: \
              name: mongodb-configmap \
              key: database_host
   to
- name: ME_CONFIG_MONGODB_URL \
  value: mongodb://mongo-0.mongo,mongo-1.mongo,mongo-2.mongo:27017/?replicaSet=rs0
  
   **Make sure you remove value from config map**

9. Deploy your web-app.yaml
10. Deploy your ingress controller and access your application using ingress
