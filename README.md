# ocd-configmap

The generic chart lets you create a config map of environment variables that you can mount into a deployment. The idea is that you can use Helmfile configuration in git to cookie cutter dozens of config maps for your microservices using this single chart. You can configure it using helmfile with something like:

```
releases:
  - name: {{ requiredEnv "ENV_PREFIX" }}-myshared-config
    labels:
      configmap: myshared
    chart: ocd-meta/ocd-configmap
    version: "1.0.0-M1"
    values: 
      - name: myshared
      - env_vars: 
        - key: ENV_VAR_ONE
          value: 'hello'
        - key: ENV_VAR_TWO
          value: 'world'
```          

The configmap can then be mounted into an `ocd-deploy` release using:

```
      - configmaps:
        - name: myshared
```

The list of names is then evaluated into a list of `envFrom` settings on a deployment. The `envFrom` setting support adding a prefix to all keys that you can set with:

```
      - configmaps:
        - name: myshared
          prefix: 'FALLBACK_'
```

Note that `ocd-deployer` lets you add env vars directly to your deployment so app specific config doesnt need to be broken out into a seperate `configmap`. Also OpenShift mounts many env vars into all pods automatially and also updates it's own DNS with details of all `service` objects in the same namespace. That means that most glue config is automatic. This means that in practice we don't commonly find the need for shared `configmaps`. We do very commonly find a need for shared secrets for microservices (e.g., API credentials to 3rd party services) created with the `ocd-secret` chart. 

There is one advantage of `configmap` over a secret. If you update a `configmap` the new values are immediately visiable to pods. If you update a `secret` you have to bounce the pods by triggering a redeployment. This means that `configmap`s are very helpful for dynamic reloading of settings witout bouncing pods. So they are something to keep in mind. 

See [ocd-meta wiki](https://github.com/ocd-scm/ocd-meta/wiki)
