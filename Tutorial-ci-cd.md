# Tutorial DevOps CI/CD

Ref:
- https://www.redpill-linpro.com/techblog/2018/11/09/openshift-jenkins-pipeline.html
- http://v1.uncontained.io/playbooks/continuous_delivery/ci-cd-elements.html#immutable-infrastructure
- https://medium.com/@dale.bingham_30375/setup-sonarqube-in-minishift-for-scanning-projects-through-jenkins-a70a6e2d93d3


#1. login Openshift/OKD Cluster as admin
```
oc login -u admin -p password https://console.i3datacenter.com:8443
```

#2. Create Project Dev, Test and Prod
```
oc new-project production
oc new-project testing
oc new-project development
```

#3. Setting Permission inter project
```
#running this command on development project
oc policy add-role-to-group system:image-puller system:serviceaccounts:production
oc policy add-role-to-group system:image-puller system:serviceaccounts:testing
```

#4. Populate development project
```
oc new-app https://github.com/kaoloon/python-flask-alan.git

#check status deployment
oc status

#expose route
oc expose svc python-flask-alan

#access the url on the web browser
```

#5. Populate Testing & Production project
```
#switch to testing project
oc project testing
oc tag development/python-flask-alan:latest python-flask-alan:test
oc new-app --image-stream=python-flask-alan:test
oc status
oc expose svc python-flask-alan

#switch to production
oc project production
oc tag development/python-flask-alan:latest python-flask-alan:prod
oc new-app --image-stream=python-flask-alan:prod
oc status
oc expose svc python-flask-alan
```

#6. Starting Jenkins
```
#create project cicd
oc new-project cicd
oc -n development policy add-role-to-user edit system:serviceaccount:cicd:jenkins
oc -n testing policy add-role-to-user edit system:serviceaccount:cicd:jenkins
oc -n production policy add-role-to-user edit system:serviceaccount:cicd:jenkins

#set pipeline
oc new-app https://github.com/kaoloon/python-flask-alan.git#pipeline

#check logs
oc logs -f bc/python-flask-alan
info: logs available at https://jenkins-cicd.apps.i3datacenter.com/blue/organizations/jenkins/cicd%2Fcicd-python-flask-alan/detail/cicd-python-flask-alan/1/
```

