https://github.com/tancheeyong/karaf2-cxf-rest.git

http://s2i-karaf2-cxf-rest-route-development.apps.rhpds.openshift.opentlc.com/cxf/crm?_wadl
http://s2i-karaf2-cxf-rest-route-development.apps.rhpds.openshift.opentlc.com/cxf/crm/customerservice/customers/123
http://s2i-karaf2-cxf-rest-production.apps.rhpds.openshift.opentlc.com/cxf/crm?_wadl
http://s2i-karaf2-cxf-rest-production.apps.rhpds.openshift.opentlc.com/cxf/crm/customerservice/customers/123

oc delete bc,dc,svc,route -l app=s2i-karaf2-cxf-rest -n uat
oc delete imagestreams/s2i-karaf2-cxf-rest -n uat
oc delete bc,dc,svc,route -l app=s2i-karaf2-cxf-rest -n production
oc delete imagestreams/s2i-karaf2-cxf-rest -n production

for ((i=1;i<=1000;i++)); do curl http://s2i-karaf2-cxf-rest-route-development.apps.rhpds.openshift.opentlc.com/cxf/crm/customerservice/customers/123 -w "\n"; sleep 3; done

while true; do echo $( curl -s http://myapp-development.192.168.99.100.nip.io/services/helloservice/sayHello/Ivan)  ;sleep 1; done

oc get hpa --watch

50 delay
100/65/30
time 1200

node {
   stage ('Deploy To UAT') {
               //timeout(time:5, unit:'MINUTES') {
                //  input message: "Promote to Production?", ok: "Promote"
               //}
               def FROM = "development"
               def TO = "uat"

               // clean up. keep the imagestream
               //sh "oc delete bc,dc,svc,route,imagestreams -l app=s2i-karaf2-cxf-rest -n ${TO}"
               //sh "oc delete imagestreams/s2i-karaf2-cxf-rest -n ${TO}"
               // deploy stage image
               sh "oc new-app ${FROM}/s2i-karaf2-cxf-rest:latest -n ${TO}"
               sh "oc patch svc/s2i-karaf2-cxf-rest -p '[{\"op\": \"replace\", \"path\": \"/spec/ports\", \"value\":[{\"name\": \"8778-tcp\",\"port\": 8778,\"protocol\": \"TCP\",\"targetPort\": 8181}]}]' --type=json -n ${TO}"
               sh "oc expose svc/s2i-karaf2-cxf-rest -n ${TO}"
             }
   stage ('Deploy To Production') {
               timeout(time:30, unit:'MINUTES') {
                  input message: "Promote to Production?", ok: "Promote"
               }
               def FROM = "development"
               def TO = "production"

               // clean up. keep the imagestream
               //sh "oc delete bc,dc,svc,route,imagestreams -l app=s2i-karaf2-cxf-rest -n ${TO}"
               //sh "oc delete imagestreams/s2i-karaf2-cxf-rest -n ${TO}"
               // deploy stage image
               sh "oc new-app ${FROM}/s2i-karaf2-cxf-rest:latest -n ${TO}"
               sh "oc patch svc/s2i-karaf2-cxf-rest -p '[{\"op\": \"replace\", \"path\": \"/spec/ports\", \"value\":[{\"name\": \"8778-tcp\",\"port\": 8778,\"protocol\": \"TCP\",\"targetPort\": 8181}]}]' --type=json -n ${TO}"
               sh "oc expose svc/s2i-karaf2-cxf-rest -n ${TO}"
             }
}

oc policy add-role-to-user edit -z cheetan-redhat.com -n production
oc policy add-role-to-user edit -z jenkins -n development
oc policy add-role-to-user edit -z jenkins -n production
oc policy add-role-to-user edit system:serviceaccount:development:jenkins -n production
oc policy add-role-to-user edit system:serviceaccount:ci:jenkins -n production
oc policy add-role-to-user edit system:serviceaccount:uat:jenkins -n production

oc new-app development/s2i-karaf2-cxf-rest:latest -n production
oc patch svc/s2i-karaf2-cxf-rest -p '[{\"op\": \"replace\", \"path\": \"/spec/ports\", \"value\":[{\"name\": \"8778-tcp\",\"port\": 8778,\"protocol\": \"TCP\",\"targetPort\": 8181}]}]' --type=json -n production
oc expose svc/s2i-karaf2-cxf-rest -n production
