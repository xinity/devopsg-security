== Secrets using Kubernetes Secrets

In this section we will demonstrate how to place secrets into the Kubernetes cluster and then show multiple ways of retrieving those secretes from within a pod.

=== Create secrets

First encode the secrets you want to apply, for this example we will use the username `admin` and the password `password`

    echo -n "admin" | base64
    echo -n "password" | base64

Both of these values are already written in the file `./templates/secret.yaml`. The configuration looks like:

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: cGFzc3dvcmQ=
```

You can now insert this secret in the Kubernetes cluster with the following command:

  kubectl apply -f ./templates/secret.yaml

The list of created secrets can be seen as:

  $ kubectl get secrets
  NAME                  TYPE                                  DATA      AGE
  default-token-4cqsx   kubernetes.io/service-account-token   3         8h
  mysecret              Opaque                                2         6s

The values of the secret are displayed as `Opaque`.

Get more details about the secret:

  $ kubectl describe secrets/mysecret
  Name:         mysecret
  Namespace:    default
  Labels:       <none>
  Annotations:  <none>

  Type:  Opaque

  Data
  ====
  password:  8 bytes
  username:  5 bytes

Once again, the values of the secret are not shown.

=== Consume in a pod volume

Deploy the pod:

    kubectl apply -f ./templates/pod-secret-volume.yaml

The pod configuration file looks like:

    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-secret-volume
    spec:
      containers:
      - name: pod-secret-volume
        image: redis
        volumeMounts:
        - name: foo
          mountPath: "/etc/foo"
          readOnly: true
      volumes:
      - name: foo
        secret:
          secretName: mysecret

Open a shell to the pod to see the secrets:

    kubectl exec -it pod-secret-volume /bin/bash
    ls /etc/foo
    cat /etc/foo/username ; echo
    cat /etc/foo/password ; echo

The above commands should result in the plain text values, the decoding is done for you.

Delete the pod:

    kubectl delete -f ./templates/pod-secret-volume.yaml

=== Consume as pod environment variables

Deploy the pod:

    kubectl apply -f ./templates/pod-secret-env.yaml

The pod configuration file looks like:

    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-secret-env
    spec:
      containers:
      - name: pod-secret-env
        image: redis
        env:
          - name: SECRET_USERNAME
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: username
          - name: SECRET_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysecret
                key: password
      restartPolicy: Never

Open a shell to the pod to see the secrets:

    kubectl exec -it pod-secret-env /bin/bash
    echo $SECRET_USERNAME
    echo $SECRET_PASSWORD

The above commands illustrate how to see the secret values via environment variables.
