### 環境
* macOS Big Sur 11.6.8
* CPU 1.3GHz Intel Core i5
* RAM 4 GB 1600 MHz DDR3
* docker (v20.10.17)
* kubernetes (v1.25.0)

### pack spring boot & build image, then push image to docker hub  
* Already upload
```
$ cd springio-api
$ docker build -t springio-demo .

# this image tag and "image" (in k8s-app/deployment.yaml) should be identical
#
$ docker login
$ docker tag springio-demo sunyayang/springio-demo:1.0.0
$ docker push sunyayang/springio-demo:1.0.0
```

### check k8s cluster status  
```
$ kubectl cluster-info
$ kubectl get nodes
$ kubectl get services
```

### apply all service (app & mysql)  
* secret type :
![image](https://user-images.githubusercontent.com/22904639/194242722-d1ebeef9-7256-47a9-9075-0e04a23bb015.png)

```
$ cd ..
//建立namespace & PVC
$ kubectl apply -f base
//起mysql
$ kubectl apply -f mysql
//起APP
$ kubectl apply -f springio-api/k8s-app
```

### check service,pod status  
* 可查詢到POD_NAME
```
$ kubectl get svc,pods -n springio
```

### or using kubernetes proxy
* -n 指定namespace
```
# open browser: http://localhost:8080/swagger-ui.html
* 開proxy才能用browser連到網頁
$ kubectl port-forward -n springio svc/springio-demo 8080:8080
```

========================== reset ===============================
### delete all in namespace "springio" & PV  
```
$ kubectl delete namespaces springio
$ kubectl delete pv mysql-pv --grace-period=0 --force
```

========================== debug ===============================
### debug pod & check pod logs  
```
$ kubectl describe pods ${POD_NAME} -n springio 
$ kubectl logs ${POD_NAME} -n springio   
```

### look inside mysql  
* 進入mysql container 密碼反編譯:root
```
$ kubectl -n springio exec -it ${MYSQL_POD_NAME} -- mysql -u root -p
```

### if pods stuck in terminating status  
* 備用指令
```
$ kubectl delete pod <PODNAME> --grace-period=0 --force -n springio  
```
