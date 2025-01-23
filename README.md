# Kustomize Tutorial

Kustomize is a tool that lets you create customized Kubernetes configuration files for your different environments in a customized way. 

## Folder structure

Generally there are two important directories that one needs to understand - `base` and `overlays`. A `base` directory consists of common kubernetes manifests across all environments. However, the base kubernetes manifests needs to be customized according to specific needs in order to get deployed to the environments. For eg. Your `base` directory can consist of a deployment stating the replicas as 3. However, for your `dev` environment, you need the replicas as `5` and for `prod`, you need the replicas as 10.

So, `base` kubernetes manifests are like a template that consists of common portion that is applicable to all environment such as name, apiVersion, kind etc. 

The another folder is the `overlays` folder that is going to consist of configurations for your specific environment such as dev, prod etc. 

## How Kustomize works

Kustomize generally looks for a file named `kustomization.yaml` that tells the Kustomize to manage the following resources. For eg. you can see that in the `base` and `overlays` folder, we have a file named `kustomization.yaml`. What happens typically is you define your common kubenretes manifests and `kustomization.yaml` in the `base` folder.

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

The above content is of the `kustomization.yaml` from the `base` folder. All it is telling the Kustomize to look for the resources defined in the files `deployment.yaml` and `service.yaml`. 

You can see that the `overlays` folder consists different folders such as `dev` and `prod` with each sub-folder consisting its own `kustomization.yaml` file. For eg. 

The `kustomization.yaml` for `dev` says this:

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

patches:
- path: deployment.yaml
- path: service.yaml
```

**IMPORTANT:** When you apply the command `kustomize build overlays/dev` Kustomize starts looking for the `kustomization.yaml` file in the `dev` folder. It sees the above contents of the file. It gets to know that it needs to manage the resources defined in the `kustomization.yaml` file of the base directory which are `deployment.yaml` and `service.yaml`. Now, it knows which resources it needs to customize for the dev environment. The next line asks to apply a patch on the base configuration file by file named `deployment.yaml` and `service.yaml`. Hence, it looks the content of the `deployment.yaml` file of the dev and applies it at the top of the `deployment.yaml` of base configuration. Similarly goes for the `service.yaml` file. For `dev` environment, we need only a deployment with 2 replicas, a Nodeport service, and less memory and CPU resources while for `prod` we need a deployment with 4 replicas, different CPU and memory limits, a rolling update strategy, and a service without NodePort.Hence, 

- base + overlays/dev = customized kubernetes manifest for dev

- base + overlays/prod = customized kubernetes manifest for prod

## References:
- https://devopscube.com/kustomize-tutorial/
- https://medium.com/@giorgiodevops/kustomize-use-patches-to-add-or-override-resources-48ef65cb634c
- https://github.com/kubernetes-sigs/kustomize/blob/master/examples/patchMultipleObjects.md
- https://kubectl.docs.kubernetes.io/guides/example/multi_base/