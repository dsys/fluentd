# fluentd

Simple log collector for Kubernetes clusters, with Loggly and S3 integrations.

## Usage

Available on Docker Hub as [pavlov/fluentd](https://hub.docker.com/r/pavlov/fluentd).

This container is meant to replace or augment cluster-level logging in Kubernetes. Accordingly, many of the directives in `fluent.conf` are borrowed from the official [fluentd image](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/fluentd-elasticsearch/fluentd-es-image). If you don't want to configure which fluentd plugins you're using (i.e. remove Loggly or change how the logs are persisted in S3), fork this repository.

### Example DaemonSet for AWS

```yaml
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: fluentd
data:
  loggly: dHJ5YWdhaW5pbnRlcm5ldHBlcnNvbg==
  aws.key: bmljZXRyeW5zYSE=
  aws.secret: bm9wZXRoaXNvbmVpc24ndHJlYWxlaXRoZXI=
  aws.s3.bucket: dHJvbG9sb2xvbG8=
  aws.s3.region: aWxpa2V0dXJ0bGVz
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: pavlov/fluentd
        imagePullPolicy: Always
        env:
        - name: FLUENTD_LOGGLY_TOKEN
          valueFrom:
            secretKeyRef:
              name: fluentd
              key: loggly
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: fluentd
              key: aws.key
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: fluentd
              key: aws.secret
        - name: FLUENTD_S3_BUCKET
          valueFrom:
            secretKeyRef:
              name: fluentd
              key: aws.s3.bucket
        - name: FLUENTD_S3_REGION
          valueFrom:
            secretKeyRef:
              name: fluentd
              key: aws.s3.region
        volumeMounts:
        - name: pos
          mountPath: /etc/fluent/pos
          readOnly: false
        - name: varlog
          mountPath: /var/log
          readOnly: false
        - name: mntephemeraldockercontainers
          mountPath: /mnt/ephemeral/docker/containers
          readOnly: true
        - name: docker-socket
          mountPath: /var/run/docker.sock
          readOnly: false
      volumes:
      - name: pos
        hostPath:
          path: /etc/fluent/pos
      - name: varlog
        hostPath:
          path: /var/log
      - name: mntephemeraldockercontainers
        hostPath:
          path: /mnt/ephemeral/docker/containers
      - name: docker-socket
        hostPath:
          path: /var/run/docker.sock
```

## Development

    $ make build
    $ make push

## License

This Docker image is released under the [BSD 3-Clause license](https://github.com/usepavlov/fluentd/blob/master/LICENSE).
