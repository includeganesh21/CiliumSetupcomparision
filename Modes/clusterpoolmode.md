I started with ipam mode to be cluster pool mode 
used nativerouting first 
for that case I have to add ipv4NativeRoutingCIDR which I used 10.8.0.0/16 and autoDirectNodeRoutes to true this installed cilium and everyting perfectly 
I execed into cilium pod and run cilium status and it showed networking as ipv4 4/254 alocated in 10.0.1.0/24 


but I was also having 
cluster health 1/14 reachable 
actule there were 14 modes in cluster 
I scheduled pods and try to get pods>58 to cschedule on simgle node but even after having replicas==160 only aroung 40 get scheduled and rest remained in pending state 
I want to get then scheduled. even resourse was available 


for this I tried removing native routing 
I remove ipv4NativeRoutingCIDR and autoDirectNodeRoutes that will afect the preformance but I need healty cluster 
I found cluster health 14/14 reachable  but this time 
in this case to I had same problem 
only around 40/254 got scheduled 
also I found our perf-app-service was not working I found our certmanager webhooks were not working and reason was applecertificate was not working and we got same error timeout for call to 10.100.0.10 which was Ip assignbed to kube-dns service in kube-system namespace 

so I again move to eni mode and want to work with ipam.mode=eni. routing =native and 
Prefix Delegation for AWS EKS

