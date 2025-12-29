# Mat3ra application containers (public)

This repository contains Apptainer (Singularity) definitions of Mat3ra public
application containers. Containers are built automatically via GitHub workflow
and are hosted on GitHub Container Registry.


## Managing ENV variables

Apptainer manages ENV variables with a number of shell scripts under
`/.singularity.d/env`:
```console
# ls -l /.singularity.d/env
total 10
-rwxr-xr-x 1 root root 1337 Aug  1 01:32 01-base.sh
-rwxr-xr-x 1 root root   85 Aug  1 01:32 10-docker2singularity.sh
-rwxr-xr-x 1 root root 1707 Dec  6 06:18 90-environment.sh
-rwxr-xr-x 1 root root    0 Dec  6 06:09 94-appsbase.sh
-rwxr-xr-x 1 root root 3052 Aug  1 01:32 95-apps.sh
-rwxr-xr-x 1 root root 1568 Aug  1 01:32 99-base.sh
-rwxr-xr-x 1 root root  922 Aug  1 01:32 99-runtimevars.sh
```
`90-environment.sh` is where the variables from `%environment` definition goes.
Above files are generated every time a new container is built, they are not
preserved from the base images. However, if we write variables to any other
file under `/.singularity.d/env`, they are preserved and sourced in runtime.
Notice the numerical prefix, that determines the order of sourcing. We can
write custom ENV variables to `$APPTAINER_ENVIRONMENT` in the `%post` section,
it will save them to `91-environment.sh`. This file will be sources runtime
automatically, if we need these variables runtime, we can source manually:
```console
if [ -f /.singularity.d/env/91-environment.sh ]; then
    . /.singularity.d/env/91-environment.sh
fi
```

1. Write ENV variables to `$APPTAINER_ENVIRONMENT` in base images.
```console
echo 'export ONEAPI_ROOT=/opt/intel-2023.1' >> $APPTAINER_ENVIRONMENT
```

or
```console
cat >> $APPTAINER_ENVIRONMENT <<'EOF'
export ONEAPI_ROOT=/opt/intel-2023.1
export TBBROOT=$ONEAPI_ROOT/tbb/2021.11
EOF
```

Notice the single quote above to preserve the bash variables in `echo` or `cat`
output, and substitute them runtime.

2. Set application container (final container) ENV variables via `%environment`
definition.

## Links
- https://apptainer.org/docs/admin/latest/
