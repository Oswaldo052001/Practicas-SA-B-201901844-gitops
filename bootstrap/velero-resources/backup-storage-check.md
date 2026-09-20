# bootstrap/velero-resources

Recursos de **configuración** de Velero (no el controlador, ese lo instala
Terraform: `P9/terraform/velero.tf` en el repo de código). Contiene dos
`Schedule`:

- `schedule-datos.yaml` — respaldo diario del namespace de la aplicación
  (`sa-p8`), incluye los volúmenes persistentes (snapshot nativo de GCE PD
  vía el plugin de GCP). Retención 30 días.
- `schedule-llave-sealed-secrets.yaml` — respaldo diario, aparte, del
  `Secret` que contiene la llave privada de Sealed Secrets
  (`kube-system`, con la etiqueta
  `sealedsecrets.bitnami.com/sealed-secrets-key=active`). Retención más
  larga (90 días): perder esta llave es más grave que perder un día de
  datos de negocio, porque vuelve indescifrable TODO lo sellado hasta ese
  momento, no solo lo de ese día.

Ambos respaldos van al mismo bucket externo (`P9/terraform-durable`), pero
como recursos de `Schedule` separados, con selectores distintos — así se
pueden restaurar independientemente (ver
`P9/terraform/scripts/restore-sealed-secrets-key.sh`, que busca
específicamente por el nombre del segundo schedule).
