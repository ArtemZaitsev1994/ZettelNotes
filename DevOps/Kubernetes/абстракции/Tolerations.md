Механизм ограничения запуска каких-либо приложений на каких-то нодах. Мы размещаем на нодах _заразу_ (taints), а на подах/приложениях размещаем _сопротивляемость_ (tolerations) этой заразе, и получается, что те поды, которые не умрут от _заразы_ на данной ноде - те и будут размещены.

### Taints
```
taints:
- effect: NoSchedule
  key: node-role.kubernetes.io/master
  value: "true"
```


### Tolerations
```
tolerations:
- effect: NoSchedule
  operator: Exists
  key: node-role.kubernetes.io/master
```