# FAQ

## selfLink was empty, can't make reference

- 原因：1.20开始`RemoveSelfLink`模式设置为了True

  RemoveSelfLink: Sets the .metadata.selfLink field to blank (empty string) for all objects and collections. This field has been deprecated since the Kubernetes v1.16 release. When this feature is enabled, the .metadata.selfLink field remains part of the Kubernetes API, but is always unset
- 修复： apiserver添加参数`- --feature-gates=RemoveSelfLink=false`
