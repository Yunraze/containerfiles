# shellcheck Container

Pretty simple: Runs `shellcheck` in a Debian container. 

Compile and run the container.

**NOTE**: For SELinux distributions, use the `:z` appendix on the volume mount to make sure SELinux doesn't block you from using the mount.

```bash
podman run --rm --volume "$PWD:/workspace:z" --workdir /workspace [CONTAINER NAME]:[TAG] /workspace/[SCRIPTNAME].sh
```

**NOTE:** Remember that the scripts will have to have `chmod +x` invoked on them, or you get an error about permission denied.

