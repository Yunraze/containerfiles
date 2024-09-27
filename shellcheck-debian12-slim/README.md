# shellcheck Container

Pretty simple: Runs `shellcheck` in a Debian container. 

Compile and run the container.

```bash
podman run --rm --volume "$PWD:/workspace:z" --workdir /workspace [CONTAINER NAME]:[TAG] /workspace/[SCRIPTNAME].sh
```

**NOTE:** Remember that the scripts will have to be `chmod +x` invoked on them, or you get an error about permission denied.

