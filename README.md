Para configurar el vamos primero necesitar ejeccutar una secuencia de tareas que se resume en:
1. Crear el proyecto.
2. Configurar los nodos infra para EgressIP.
3. asignar EgressIP a los pods en los nodos worker.
4. crear el servicio para la base de datos Oracle.
5. Definir la política de red necesaria.

oc new-project prueba-egress

oc label node infra-0.ocp02.promnet.com.sv k8s.ovn.org/egress-assignable=""
oc label node infra-1.ocp02.promnet.com.sv k8s.ovn.org/egress-assignable=""

apiVersion: k8s.ovn.org/v1
kind: EgressIP
metadata:
  name: egressip-prueba
spec:
  egressIPs:
    - 10.5.248.57
    - 10.5.248.58
  namespaceSelector:
    matchLabels:
      name: prueba-egress
  podSelector:
    matchLabels:
      role: worker

apiVersion: v1
kind: Service
metadata:
  name: oracle-db
  namespace: prueba-egress
spec:
  type: ExternalName
  externalName: 10.100.100.177
  ports:
    - protocol: TCP
      port: 1521
      targetPort: 1521

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-oracle-db
  namespace: prueba-egress
spec:
  podSelector:
    matchLabels:
      role: worker
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.100.100.177/32
      ports:
        - protocol: TCP
          port: 1521
