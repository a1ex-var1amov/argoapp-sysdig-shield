# argoapp-sysdig-shield


```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sysdig-shield-eks
  namespace: argocd
spec:
  destination:
    namespace: sysdig-agent
    server: https://kubernetes.default.svc
  ignoreDifferences:
  - jsonPointers:
    - /data
    kind: Secret
    name: sysdig-shield-cluster
  - jsonPointers:
    - /data
    kind: Secret
    name: sysdig-shield-cluster-tls-certificates
  - group: admissionregistration.k8s.io
    jqPathExpressions:
    - .webhooks[].clientConfig.caBundle
    kind: ValidatingWebhookConfiguration
  - group: apps
    jsonPointers:
    - /spec/template/metadata/annotations/checksum~1webhook
    kind: Deployment
    name: sysdig-shield-cluster
  project: default
  source:
    chart: shield
    helm:
      releaseName: sysdig
      values: |-
        cluster_config:
          name: 'eks'
        sysdig_endpoint:
          region: us2
          access_key: <REDACTED>
          secure_api_token: <REDACTED>
        features:
          admission_control:
            enabled: false
          kubernetes_metadata:
            enabled: true
          posture:
            host_posture:
              enabled: true
            cluster_posture:
              enabled: true
          vulnerability_management:
            host_vulnerability_management:
              enabled: true
            container_vulnerability_management:
              enabled: true
            in_use:
              enabled: true
              integration_enabled: true
          detections:
            drift_control:
              enabled: true
            malware_control:
              enabled: true
            ml_policies:
              enabled: true
            kubernetes_audit:
              enabled: true
          investigations:
            activity_audit:
              enabled: true
            live_logs:
              enabled: true
            network_security:
              enabled: true
            audit_tap:
              enabled: true
            captures:
              enabled: true
        host:
          driver: universal_ebpf
          privileged: true
        cluster:
          replica_count: 1
          additional_settings:
            log_level: debug
    repoURL: https://charts.sysdig.com/
    targetRevision: ^0
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
```
